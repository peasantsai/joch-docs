# KnowledgeSource

A `KnowledgeSource` points to a corpus that feeds a [`RAG`](rag.md) index — a file tree, a URL, an S3 bucket, a database, a connector, or another corpus type. Joch tracks knowledge sources as first-class records so ABOM, governance, and compliance can answer "where does this agent's knowledge come from?"

[Back to the catalog](index.md)

## Spec and status

```yaml
apiVersion: joch.dev/v1alpha1
kind: KnowledgeSource
metadata:
  name: support-docs-public
  namespace: support-platform
spec:
  description: Public support documentation, indexed nightly.

  type: web

  connector:
    secretRef:
      name: web-connector

  source:
    url: https://example.com/help
    crawl:
      maxDepth: 3
      includePatterns: ["/help/*"]
      excludePatterns: ["/help/internal/*"]

  sync:
    mode: incremental
    interval: 1h

  classification:
    sensitivity: public
    retention:
      days: 365

  permissions:
    mode: source_inherited

  transforms:
    - extract_text
    - preserve_headings
    - detect_tables

  policies:
    - name: no-private-doc-leakage

status:
  phase: Ready
  documentsFound: 4128
  lastSyncAt: "2026-05-09T22:00:00Z"
  bytes: 412Mi
```

## Source types

```text
web              crawl a URL with allow / deny patterns
file             ingest a local or mounted directory
s3               ingest from an S3 bucket
gcs              ingest from a GCS bucket
google_drive     ingest from a Google Drive scope
http-archive     ingest from a versioned archive
database         pull from a SQL or NoSQL source via adapter
api              pull from a paginated API
```

## Classification

`classification.sensitivity` propagates into ABOM and policies so PII / customer-tier / confidential corpora can be governed differently from public ones. Classifications: `public`, `internal`, `customer-tier`, `pii`, `regulated`.

[Back to the catalog](index.md)
