# State Portability

## Core principle

Treat OpenAI, Claude, and other vendors as **execution backends**, not as owners of state.

So instead of this:

```text
Conversation lives inside OpenAI thread
↓
Need to switch to Claude
↓
Try to translate OpenAI thread directly
```

I would do this:

```text
joch-owned state store
↓
Provider adapter renders state for OpenAI
↓
Provider adapter renders same state for Claude
```

That means `joch` owns:

```yaml
conversation_id: conv_123
agent_id: researcher
model_backend: openai:gpt-5.1
messages:
  - role: user
    content:
      - type: text
        text: "Research agent fleet managers"
  - role: assistant
    content:
      - type: text
        text: "Here are the key findings..."
    tool_calls:
      - id: call_1
        tool: web.search
        args:
          query: "agent fleet management"
    metadata:
      provider: openai
      model: gpt-5.1
memory_refs:
  - mem://project/joch/market-notes
artifacts:
  - artifact://report/market-analysis-v1
policies:
  allowed_tools:
    - web.search
    - github.read
    - filesystem.write
```

Then the OpenAI adapter and Claude adapter translate this into their own message formats.

## The hard part: state is not just messages

A mid-conversation switch has at least five state layers:

1. **Message history**
2. **Tool call history**
3. **Working memory**
4. **Long-term memory**
5. **Execution state**

The mistake would be to only migrate chat messages. That loses too much.

I would split state like this:

```text
Conversation transcript
  Durable, append-only, vendor-neutral

Working memory
  Summaries, task state, scratchpad, active goals

Long-term memory
  Vector/RAG stores, user preferences, project knowledge

Tool state
  Tool calls, outputs, permissions, side effects

Runtime state
  Current step, retries, pending approvals, locks
```

The transcript is important, but the **working memory** is what makes migration reliable.

## Use a canonical message format

I would define an internal format similar to:

```ts
type AgentMessage =
  | {
      role: "system" | "user" | "assistant" | "tool";
      content: ContentBlock[];
      toolCalls?: ToolCall[];
      toolResults?: ToolResult[];
      vendorMetadata?: Record<string, unknown>;
    };

type ContentBlock =
  | { type: "text"; text: string }
  | { type: "image"; uri: string; mimeType: string }
  | { type: "file"; uri: string; mimeType: string }
  | { type: "reasoning_summary"; text: string };

type ToolCall = {
  id: string;
  name: string;
  args: Record<string, unknown>;
};

type ToolResult = {
  callId: string;
  status: "success" | "error";
  content: ContentBlock[];
};
```

Important: I would preserve vendor-specific metadata, but never depend on it.

```yaml
vendorMetadata:
  openai:
    response_id: resp_abc
    model: gpt-5.1
  anthropic:
    message_id: msg_xyz
```

That allows debugging and replay, while keeping the actual state portable.

## Tool calls need special treatment

Tool calls are one of the biggest migration hazards.

Different vendors represent tool use differently. So I would normalize tool calls into an `joch` event log:

```text
event: tool.call.requested
tool: github.search_issues
args: { repo: "org/joch", query: "memory migration" }

event: tool.call.completed
result_ref: artifact://tool-results/abc123
status: success
```

When switching from OpenAI to Claude, Claude does not need to replay the tool call unless necessary. It can receive a compact summary:

```text
Earlier in the task, the agent called github.search_issues with query X.
The result was Y.
The relevant artifact is available at artifact://tool-results/abc123.
```

This prevents duplicate side effects.

For non-idempotent tools, I would mark them explicitly:

```yaml
tool_call:
  id: call_123
  idempotent: false
  side_effects:
    - sent_email
    - created_ticket
```

The new backend must not repeat those unless a policy permits it.

## I would introduce a migration checkpoint

When switching providers, `joch` should not simply dump the full conversation into Claude. It should create a **migration checkpoint**:

```yaml
checkpoint_id: chk_456
conversation_id: conv_123
from_backend: openai:gpt-5.1
to_backend: anthropic:claude-sonnet
summary:
  user_goal: "Design state persistence for joch"
  current_task: "Explain migration strategy"
  decisions_made:
    - "Use vendor-neutral state as source of truth"
    - "Persist tool calls as event log"
    - "Avoid replaying side-effectful tools"
  open_questions:
    - "How much vendor metadata should be retained?"
  relevant_memory_refs:
    - mem://project/joch/state-model
  active_artifacts:
    - artifact://design/state-persistence-draft
```

Then the Claude adapter receives:

```text
System/personality config
+ migration checkpoint
+ recent message window
+ relevant memory snippets
+ tool registry
```

Not necessarily the entire raw history.

## Context window strategy

A provider switch may also mean switching context window size, tokenization, or modality support.

So I would use a tiered context reconstruction strategy:

```text
1. Always include system/personality/policy config
2. Include migration checkpoint
3. Include active task state
4. Include last N turns
5. Retrieve relevant long-term memory
6. Attach artifacts by reference
7. Include older transcript only if needed
```

This gives the new model enough continuity without requiring a brittle one-to-one transcript migration.

## Personality/state separation

For `joch`, I would keep these separate:

```text
personality.md     → how the agent behaves
mission.md         → what the agent is trying to accomplish
memory store       → what the agent knows
conversation log   → what has happened
runtime state      → what is currently in progress
```

That matters because the agent may switch from OpenAI to Claude, but the **agent identity** should remain stable.

So the user experiences:

```text
Same agent, different engine.
```

Not:

```text
New chatbot pretending to remember the old one.
```

## What happens during the actual switch

A command might look like:

```bash
joch agents switch researcher \
  --from openai:gpt-5.1 \
  --to anthropic:claude-sonnet \
  --conversation conv_123
```

Internally:

```text
1. Freeze current conversation state
2. Flush pending tool results
3. Generate migration checkpoint
4. Validate target model capabilities
5. Rebuild context for target provider
6. Start next turn with new backend
7. Append migration event to audit log
```

The audit log would show:

```yaml
event: backend.switched
agent: researcher
conversation: conv_123
from: openai:gpt-5.1
to: anthropic:claude-sonnet
checkpoint: chk_456
reason: cost_policy
timestamp: 2026-05-09T...
```

## Handling model capability mismatch

Provider switching is not always safe. The target backend may lack:

```text
vision
long context
JSON mode
computer use
specific tool-calling semantics
structured output guarantees
reasoning traces
```

So I would add a compatibility check:

```bash
joch agents switch researcher --to claude-sonnet --dry-run
```

Example output:

```text
Compatibility check:
✓ Text conversation supported
✓ MCP tools supported
✓ 200k context sufficient
⚠ JSON schema strict mode differs
⚠ Previous model used image input; target model supports images but adapter must transform format
✗ Computer-use tool unavailable
```

Then policies decide whether to allow the switch.

## Storage model

I would probably persist state using an event-sourced model:

```text
ConversationEvents
AgentStateSnapshots
ToolCallEvents
MemoryReferences
ArtifactReferences
ProviderMetadata
```

Event sourcing is useful because we can reconstruct the state for any backend and audit what happened.

A simplified schema:

```sql
conversations(id, agent_id, created_at, current_backend)

conversation_events(
  id,
  conversation_id,
  sequence_number,
  event_type,
  payload_json,
  provider,
  model,
  created_at
)

state_checkpoints(
  id,
  conversation_id,
  summary,
  active_goals_json,
  memory_refs_json,
  artifact_refs_json,
  created_at
)

tool_events(
  id,
  conversation_id,
  tool_name,
  call_id,
  args_json,
  result_ref,
  idempotency_key,
  side_effect_level,
  created_at
)
```

## The key abstraction

I would make the central API something like:

```ts
interface AgentBackendAdapter {
  renderPrompt(state: CanonicalAgentState): ProviderRequest;
  parseResponse(response: ProviderResponse): CanonicalAgentEvent[];
  supports(capability: Capability): boolean;
}
```

Then OpenAI, Claude, Gemini, local models, and future providers become adapters.

The persisted state remains:

```text
joch-native
```

not:

```text
OpenAI-native
Claude-native
LangGraph-native
```

## The biggest design rule

**Never migrate state by asking one model to summarize itself without validation.**

That is tempting, but dangerous.

A better flow:

```text
1. Deterministic extraction from event log
2. Optional model-generated summary
3. Schema validation
4. Human/auditable checkpoint
5. Target-provider reconstruction
```

The model can help compress state, but the control plane owns the truth.
