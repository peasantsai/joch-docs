# Artifact

An `Artifact` is a durable output of an [`Execution`](execution.md): a generated report, dataset, file, image, transcript, or any payload that should be retained beyond the run. Artifacts are referenced by other resources (`ToolCall.output`, `Conversation` content blocks, `Trace` events) instead of being inlined.

[Back to the catalog](index.md)

## Spec and status

```yaml
apiVersion: joch.dev/v1alpha1
kind: Artifact
metadata:
  name: support-77123-draft-reply
  namespace: support-platform
spec:
  type: document

  producedBy:
    executionRef: { name: exec-20260510-001 }
  agentRef: { name: support-triage }

  storage:
    backend: s3
    bucket: joch-artifacts-prod
    keyTemplate: "{{ namespace }}/{{ agent }}/{{ execution }}/{{ name }}.md"
    encryption:
      type: kms
      keyRef:
        name: artifact-kms-key

  content:
    mimeType: text/markdown
    classification: customer-tier
    sizeBytes: 2148
    sha256: "f3c1..."

  provenance:
    sources:
      - artifact://kb/support-docs-public/refund-policy.md
    toolCalls:
      - call-abc123

  retention:
    days: 90

status:
  phase: Ready
  uri: artifact://support-platform/support-triage/exec-20260510-001/support-77123-draft-reply.md
  storedAt: "2026-05-10T10:35:13Z"
```

## Backends

```text
local          local filesystem (developer mode)
s3             S3-compatible object store
gcs            Google Cloud Storage
minio          MinIO (S3-compatible)
azure-blob     Azure Blob Storage
```

The `Artifact` resource is the same shape across backends.

## Why separate from Trace

Trace events are dense and high-cardinality; artifacts are large and low-cardinality. Keeping them in different stores keeps trace queries fast and artifact retention cost-effective.

## Provenance

`provenance` makes artifacts auditable: which sources informed them, which tool calls produced them. This feeds compliance and reproducibility for downstream consumers.

[Back to the catalog](index.md)
