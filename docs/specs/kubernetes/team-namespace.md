# Team / Namespace

Kubernetes-style YAML resource specification for Joch `Team / Namespace` resources.

[Back to Kubernetes specs](index.md)

## Team / Namespace spec

Eventually we will need ownership.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Team
metadata:
  name: growth
spec:
  members:
    - user: alice@example.com
      role: admin
    - user: bob@example.com
      role: operator

  namespaces:
    - growth-dev
    - growth-prod

  defaultPolicies:
    - budget-policy
    - pii-policy
```

---
