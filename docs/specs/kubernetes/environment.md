# Environment

Kubernetes-style YAML resource specification for Joch `Environment` resources.

[Back to Kubernetes specs](index.md)

## Environment spec

```yaml
apiVersion: joch.dev/v1alpha1
kind: Environment
metadata:
  name: prod-eu
spec:
  region: eu-central

  runtime:
    orchestrator: kubernetes
    cluster: prod-agent-cluster

  defaults:
    modelRef:
      name: gpt-5-thinking
    traceRetentionDays: 30
    logLevel: info

  compliance:
    dataResidency: EU
    piiMode: redact
    auditRequired: true
```

This lets we do:

```bash
joch deploy research-agent --env prod-eu
```

---
