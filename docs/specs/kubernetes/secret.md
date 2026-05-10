# Secret

Kubernetes-style YAML resource specification for Joch `Secret` resources.

[Back to Kubernetes specs](index.md)

## Secret spec

Joch can either wrap Kubernetes secrets or define its own reference object.

```yaml
apiVersion: joch.dev/v1alpha1
kind: SecretRef
metadata:
  name: openai-api-key
spec:
  provider: vault
  path: secret/data/openai
  key: apiKey
```

I would avoid storing actual secrets in `joch` specs.

---
