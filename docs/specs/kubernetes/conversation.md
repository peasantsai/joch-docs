# Conversation

Kubernetes-style YAML resource specification for Joch `Conversation` resources.

[Back to Kubernetes specs](index.md)

## Add `Conversation` as a resource

For vendor-agnostic state persistence, this is essential.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Conversation
metadata:
  name: conv-123
spec:
  agentRef:
    name: research-agent

  stateStore:
    type: postgres
    ref: conversation-store-prod

  retention:
    maxDays: 90

  summarization:
    enabled: true
    checkpointEveryMessages: 20
    modelRef:
      name: gpt-5-mini

  portability:
    canonicalFormat: joch.dev/conversation.v1
    preserveVendorMetadata: true

status:
  phase: Active
  messageCount: 82
  currentBackend: openai:gpt-5-thinking
  lastCheckpoint: chk-456
```

---
