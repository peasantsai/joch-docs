# Conversation

A `Conversation` is a vendor-neutral, durable record of an agent's dialog. It carries the canonical message history, tool-call event log, memory references, and artifact references — and it survives changes of model provider via [`StateCheckpoint`](state-checkpoint.md).

[Back to the catalog](index.md)

## Spec and status

```yaml
apiVersion: joch.dev/v1alpha1
kind: Conversation
metadata:
  name: conv-77123
  namespace: support-platform
spec:
  agentRef: { name: support-triage }

  stateStore:
    type: postgres
    ref: conversation-store-prod

  retention:
    maxDays: 90

  summarization:
    enabled: true
    checkpointEveryMessages: 20
    summarizerModelRef: { name: gpt-5-mini }

  portability:
    canonicalFormat: joch.dev/conversation.v1
    preserveVendorMetadata: true

status:
  phase: Active
  messageCount: 82
  currentBackend: openai:gpt-5-thinking
  lastCheckpointRef: { name: chk-456 }
```

## Why a separate resource

Vendor SDKs treat conversations as ephemeral threads tied to a provider. Joch persists conversations in a vendor-neutral form so that:

- migrations between providers preserve dialog continuity,
- audits can replay any conversation,
- multiple agents can share a conversation through handoffs,
- conversation summaries can be regenerated without re-running the agent.

## Canonical message format

See [State Portability](../../architecture/state-portability.md) for the canonical message and tool-call event types. The `Conversation` is the storage envelope around that event log.

## Summarization

`summarization` controls when Joch creates a [`StateCheckpoint`](state-checkpoint.md) — periodically by message count, or on demand before a provider switch. Summaries are deterministic extractions plus optional model-generated additions, validated against schema.

## Portability

`portability.canonicalFormat` declares the version of the Joch conversation schema in use. `preserveVendorMetadata` retains provider-specific fields (e.g., OpenAI `response_id`, Anthropic `message_id`) for debugging — but never as the source of truth.

[Back to the catalog](index.md)
