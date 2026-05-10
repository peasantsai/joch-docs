# State Portability

> Treat OpenAI, Anthropic, Google, Microsoft Foundry, and local model servers as **execution backends**, not as owners of state.

State portability is the architectural principle that makes the [Portability pillar](../pillars/portability.md) possible. The control plane owns conversation, memory, tool, and runtime state. Provider adapters render that state into each vendor's request format. The persisted state is always Joch-native.

## Layers of state

A mid-conversation provider switch must preserve five distinct layers, not just the chat transcript:

```text
1. Message history       what was said
2. Tool-call history     what was done
3. Working memory        scratchpad / task state / active goals
4. Long-term memory      vector / RAG / preferences / project knowledge
5. Runtime state         current step / retries / pending approvals / locks
```

Migrating only chat messages destroys the other four layers. Joch keeps all five durable.

## Canonical message format

The internal format is intentionally simple:

```ts
type AgentMessage = {
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

`vendorMetadata` is preserved but never depended on:

```yaml
vendorMetadata:
  openai:
    response_id: resp_abc
    model: gpt-5.5-thinking
  anthropic:
    message_id: msg_xyz
```

This preserves debug fidelity without coupling the canonical state to a vendor.

## Tool-call state

Tool calls are the single biggest hazard during a switch. Joch persists every tool call as an event:

```text
event: tool.call.requested
tool: github.search_issues
args: { repo: "org/joch", query: "memory migration" }

event: tool.call.completed
result_ref: artifact://tool-results/abc123
status: success
side_effects: read_only
```

When a switch happens, the new provider does **not** replay tool calls. It receives a compact summary:

```text
Earlier in the task, the agent called github.search_issues with query X.
The result was stored as artifact://tool-results/abc123.
```

Side-effecting calls are flagged so the new provider cannot accidentally repeat them:

```yaml
toolCall:
  id: call_123
  idempotencyKey: zendesk-create-12345
  sideEffects:
    level: external_write
    idempotent: false
```

## Migration checkpoint

A provider switch creates a deterministic [`StateCheckpoint`](../specs/kubernetes/state-checkpoint.md):

```yaml
apiVersion: joch.dev/v1alpha1
kind: StateCheckpoint
metadata:
  name: chk-456
spec:
  conversationRef: { name: conv-123 }
  fromBackend: openai:gpt-5-thinking
  toBackend: anthropic:claude-sonnet
  summary:
    userGoal: "Design state persistence for Joch"
    currentTask: "Explain migration strategy"
    decisions:
      - "Use vendor-neutral state as the source of truth"
      - "Persist tool calls as event log"
      - "Avoid replaying side-effectful tools"
    openQuestions:
      - "How much vendor metadata to retain"
  activeArtifacts:
    - artifact://design/state-persistence-draft
  relevantMemoryRefs:
    - mem://project/joch/state-model
```

The new backend then receives:

```text
system / personality / policy config
+ migration checkpoint
+ recent message window
+ relevant memory snippets
+ tool registry
```

Not the entire raw transcript.

## Tiered context reconstruction

Different providers have different context window sizes, tokenizers, and modality support. Joch reconstructs context in tiers:

```text
1. Always include system / policy / personality config
2. Include the migration checkpoint
3. Include the active task state
4. Include the last N turns
5. Retrieve relevant long-term memory
6. Attach artifacts by reference
7. Include older transcript only if budget allows and needed
```

This gives the target model continuity without forcing a brittle one-to-one transcript transfer.

## Capability validation

Provider switches are not always safe. The target backend may lack vision, structured output, computer use, long context, JSON-strict mode, or specific tool-calling semantics. Joch validates capabilities before committing:

```bash
joch agents switch researcher --to claude-sonnet --dry-run
```

```text
Compatibility check:
✓ Text conversation supported
✓ MCP tools supported
✓ 200k context sufficient
⚠ JSON schema strict mode differs (loose-parse fallback)
⚠ Previous turn used image input; image format will be transformed
✗ Computer-use tool unavailable; tool will be removed for this conversation
```

Policy decides whether warnings or failures gate the switch. See [Model Router](model-router.md).

## Storage model

Conversation state is event-sourced:

```text
ConversationEvents       append-only message + tool events
StateCheckpoints         materialized summaries used by migrations
MemoryReferences         pointers to memory store items
ArtifactReferences       pointers to artifact store items
ProviderMetadata         vendor-specific debug data, never source-of-truth
```

A simplified relational schema:

```sql
conversations (
  id,
  agent_id,
  current_backend,
  created_at
);

conversation_events (
  id,
  conversation_id,
  sequence_number,
  event_type,
  payload_json,
  provider,
  model,
  created_at
);

state_checkpoints (
  id,
  conversation_id,
  summary,
  active_goals_json,
  memory_refs_json,
  artifact_refs_json,
  created_at
);

tool_events (
  id,
  conversation_id,
  tool_name,
  call_id,
  args_json,
  result_ref,
  idempotency_key,
  side_effect_level,
  created_at
);
```

Event sourcing lets Joch reconstruct state for any backend and audit exactly what happened.

## The central abstraction

```ts
interface AgentBackendAdapter {
  renderPrompt(state: CanonicalAgentState): ProviderRequest;
  parseResponse(response: ProviderResponse): CanonicalAgentEvent[];
  supports(capability: Capability): boolean;
}
```

OpenAI, Anthropic, Google, Microsoft, and local models become adapters. The persisted state is always **Joch-native**, never `OpenAI-native`, `Claude-native`, or `LangGraph-native`.

## The largest design rule

> **Never migrate state by asking one model to summarize itself without validation.**

It is tempting and dangerous. The correct flow is:

```text
1. Deterministic extraction from the event log
2. Optional model-generated summary
3. Schema validation
4. Auditable checkpoint
5. Target-provider reconstruction
```

The model can help compress state. The control plane owns the truth.
