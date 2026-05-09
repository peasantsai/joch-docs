# Handoff

Kubernetes-style YAML resource specification for Joch `Handoff` resources.

[Back to Kubernetes specs](index.md)

## Add `Handoff` as a runtime resource

Multi-agent systems need explicit handoff records.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Handoff
metadata:
  name: handoff-001
spec:
  executionRef:
    name: exec-20260509-001

  fromAgent:
    name: research-agent

  toAgent:
    name: writer-agent

  reason: >
    Research phase complete; writer-agent should draft final report.

  context:
    summary: >
      Found 12 competing tools and 4 differentiation themes.
    artifactRefs:
      - market-research-notes
    memoryRefs:
      - research-agent-working-memory

  expectedOutput:
    type: markdown_report

status:
  phase: Accepted
```

---
