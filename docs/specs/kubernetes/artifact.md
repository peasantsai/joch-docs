# Artifact

Kubernetes-style YAML resource specification for Joch `Artifact` resources.

[Back to Kubernetes specs](index.md)

## Artifact spec

Agents produce durable things.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Artifact
metadata:
  name: market-analysis-report
spec:
  type: document
  producedBy:
    executionRef:
      name: exec-20260509-001

  storage:
    type: s3
    uri: s3://agent-artifacts/reports/market-analysis.md

  content:
    mimeType: text/markdown
    checksum: sha256:abc123

  provenance:
    sources:
      - https://example.com/source-1
    toolCalls:
      - call_123
      - call_456

status:
  phase: Ready
```

Artifacts give Joch reproducibility and auditability.

---
