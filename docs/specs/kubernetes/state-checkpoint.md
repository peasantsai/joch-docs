# StateCheckpoint

Kubernetes-style YAML resource specification for Joch `StateCheckpoint` resources.

[Back to Kubernetes specs](index.md)

## Add `StateCheckpoint`

This supports model/provider switching.

```yaml
apiVersion: joch.dev/v1alpha1
kind: StateCheckpoint
metadata:
  name: chk-456
spec:
  conversationRef:
    name: conv-123

  executionRef:
    name: exec-20260509-001

  summary:
    userGoal: Design joch resource specs.
    currentTask: Draft Kubernetes-style YAML specifications.
    decisions:
      - Use vendor-neutral state.
      - Treat tools, skills, and MCP servers separately.
      - Persist tool calls as runtime resources.
    openQuestions:
      - Which resources belong in v1alpha1?

  activeArtifacts:
    - joch-resource-spec-draft

  memoryRefs:
    - research-agent-working-memory

  providerState:
    from:
      provider: openai
      model: gpt-5-thinking
    portable: true

status:
  phase: Ready
```

---
