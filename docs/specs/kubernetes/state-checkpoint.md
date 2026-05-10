# StateCheckpoint

A `StateCheckpoint` is a vendor-neutral mid-conversation snapshot used to migrate a conversation between providers without losing tools, memory, or artifacts. Checkpoints are produced by [`Conversation`](conversation.md) summarization and consumed by the [model router](../../architecture/model-router.md) during provider switches.

[Back to the catalog](index.md)

## Spec and status

```yaml
apiVersion: joch.dev/v1alpha1
kind: StateCheckpoint
metadata:
  name: chk-456
  namespace: support-platform
spec:
  conversationRef: { name: conv-77123 }
  executionRef:    { name: exec-20260510-001 }

  fromBackend: openai:gpt-5-thinking
  toBackend:   anthropic:claude-sonnet

  summary:
    userGoal: "Resolve refund request for order 12345"
    currentTask: "Draft reply for human review"
    decisions:
      - "Customer is eligible for a refund per policy R-12"
      - "Replacement will not be offered"
    openQuestions:
      - "Confirm shipping address"
    activeArtifacts:
      - artifact://exec/exec-20260510-001/draft-reply.md
    relevantMemoryRefs:
      - mem://support/customer-77123-context

  providerState:
    from:
      provider: openai
      model: gpt-5-thinking
      vendorMetadata:
        response_id: resp_abc
    portable: true

status:
  phase: Ready
  createdAt: "2026-05-10T10:35:10Z"
  appliedToBackend: anthropic:claude-sonnet
  appliedAt: "2026-05-10T10:35:11Z"
```

## What a checkpoint is for

A checkpoint is a deterministic extraction from the conversation event log, optionally enriched with a model-generated summary, validated against the Joch schema. The target provider receives:

```text
system / personality / policy config
+ migration checkpoint
+ recent message window
+ relevant memory snippets
+ tool registry
```

— not the full raw transcript. This is the only safe way to migrate a long conversation across providers with different context windows, tokenizers, and modalities. See [State Portability](../../architecture/state-portability.md).

## Capability validation

Checkpoint application is preceded by a capability check. If the target backend lacks a required capability (vision, structured output, computer use), the policy engine decides whether to allow with adapted modality, allow with degraded capability, or deny.

## Auditability

Every applied checkpoint produces a `ProviderSwitched` event in the trace, with the from-provider, to-provider, checkpoint reference, and reason.

[Back to the catalog](index.md)
