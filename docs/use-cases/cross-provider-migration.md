# Cross-Provider Migration

> A research agent runs on OpenAI today. A new pricing tier or a regional outage forces a switch to Anthropic mid-conversation, without losing tools, memory, artifacts, or repeating side effects.

## Symptoms

- Provider outages crash long-running agent conversations.
- Cost spikes on one provider cannot be relieved without rewriting agent code.
- A regulatory request mandates EU-only data residency and the agent's primary provider has no EU region.
- A mid-conversation switch attempted naively duplicates side effects (re-creates issues, re-sends emails).

## What Joch does

Joch implements [State Portability](../architecture/state-portability.md): vendor-neutral [`Conversation`](../specs/kubernetes/conversation.md) and [`StateCheckpoint`](../specs/kubernetes/state-checkpoint.md) resources, capability validation, and side-effect-aware migration through the [`ModelRoute`](../specs/kubernetes/model-route.md) and the [Model Router](../architecture/model-router.md).

## Walkthrough

### 1. Define a route, not a model

```yaml
apiVersion: joch.dev/v1alpha1
kind: ModelRoute
metadata:
  name: research-default
spec:
  requirements:
    toolCalling: true
    contextWindowMin: 200000
    structuredOutput: true
  strategy:
    primary: openai:gpt-5-thinking
    fallback:
      - anthropic:claude-sonnet
      - google:gemini-pro
  constraints:
    maxCostUsdPerExecution: 5
    dataResidency: EU
```

Agents reference the route. Provider choice becomes an operational concern.

### 2. Dry-run the switch

```bash
joch agents switch researcher --to anthropic:claude-sonnet --conversation conv-99 --dry-run
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

Policy decides whether warnings or failures gate the switch.

### 3. Create the checkpoint

A live switch generates a deterministic [`StateCheckpoint`](../specs/kubernetes/state-checkpoint.md):

```yaml
apiVersion: joch.dev/v1alpha1
kind: StateCheckpoint
metadata: { name: chk-456 }
spec:
  conversationRef: { name: conv-99 }
  fromBackend: openai:gpt-5-thinking
  toBackend:   anthropic:claude-sonnet
  summary:
    userGoal: "Analyze competitive landscape for agent ops tools"
    currentTask: "Draft section 3 of the report"
    decisions:
      - "Five competitors in scope"
      - "Excluded SDK-only entrants"
    activeArtifacts:
      - artifact://research/landscape-draft.md
    relevantMemoryRefs:
      - mem://research/competitor-notes
```

### 4. Apply the checkpoint

```bash
joch agents switch researcher --to anthropic:claude-sonnet --conversation conv-99 --apply
```

The new provider receives **system + checkpoint + recent window + relevant memory + tool registry** — not the full raw transcript. Side-effecting tool calls already executed are summarized, not replayed.

### 5. Verify and audit

```bash
joch trace conv-99 --since 1h
joch route exec-20260510-002 --explain
joch describe statecheckpoint chk-456
```

A `ProviderSwitched` event is recorded with from-provider, to-provider, checkpoint reference, capability deltas, and reason.

## Resources involved

- [`ModelRoute`](../specs/kubernetes/model-route.md), [`Model`](../specs/kubernetes/model.md)
- [`Conversation`](../specs/kubernetes/conversation.md), [`StateCheckpoint`](../specs/kubernetes/state-checkpoint.md)
- [`ToolCall`](../specs/kubernetes/toolcall.md) (for idempotency)
- [`Trace`](../specs/kubernetes/trace.md)

## Outcome

The team treats providers as substrate. Outages, price changes, and residency requirements become route edits. Conversations survive provider switches with full continuity and no duplicated side effects.
