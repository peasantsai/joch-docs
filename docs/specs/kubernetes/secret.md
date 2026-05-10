# Secret

A `Secret` is a **reference** to an external secret. Joch never stores secret values. The reference is resolved at the data-plane gateway boundary — the model router for provider keys, the tool gateway for tool credentials, the MCP gateway for MCP server credentials.

[Back to the catalog](index.md)

## Spec and status

```yaml
apiVersion: joch.dev/v1alpha1
kind: Secret
metadata:
  name: openai-api-key
  namespace: support-platform
spec:
  source:
    type: kubernetes
    name: openai-credentials
    key: api-key

  rotation:
    policy: scheduled
    intervalDays: 90

  scope:
    boundTo:
      models:
        - openai:gpt-5-thinking
      mcpServers: []
      tools: []

status:
  phase: Ready
  lastResolvedAt: "2026-05-10T10:34:50Z"
  rotationDueAt: "2026-08-01T00:00:00Z"
```

## Source types

```text
env                  read from a process environment variable (developer mode)
kubernetes           Kubernetes Secret
vault                HashiCorp Vault path
aws-secrets-manager  AWS Secrets Manager ARN
gcp-secret-manager   Google Cloud Secret Manager
azure-key-vault      Azure Key Vault
```

The provider behind each source is a pluggable adapter.

## Why values never live in Joch

- Joch's resource store is replicated and inspected by many operators; storing values there increases blast radius.
- Resolution at the gateway boundary is a small, auditable code path.
- Rotation is the responsibility of the source-of-truth secret system; Joch follows it.

The result is a single, narrow point where credentials enter outbound calls — and never anywhere else.

[Back to the catalog](index.md)
