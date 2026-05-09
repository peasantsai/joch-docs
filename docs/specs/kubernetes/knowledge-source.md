# KnowledgeSource

Kubernetes-style YAML resource specification for Joch `KnowledgeSource` resources.

[Back to Kubernetes specs](index.md)

## KnowledgeSource spec

```yaml
apiVersion: joch.dev/v1alpha1
kind: KnowledgeSource
metadata:
  name: product-docs
spec:
  type: google_drive

  connector:
    secretRef:
      name: google-drive-connector

  include:
    paths:
      - /Product
      - /Engineering/RFCs

  exclude:
    patterns:
      - "**/Archive/**"
      - "**/*.tmp"

  sync:
    mode: incremental
    interval: 30m

  permissions:
    mode: source_inherited

  transforms:
    - extract_text
    - preserve_headings
    - detect_tables

status:
  phase: Ready
  lastSyncAt: "2026-05-09T09:45:00Z"
  documentsFound: 1220
```

This lets `joch` manage RAG pipelines cleanly.

---
