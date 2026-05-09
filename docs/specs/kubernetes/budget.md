# Budget

Kubernetes-style YAML resource specification for Joch `Budget` resources.

[Back to Kubernetes specs](index.md)

## Budget spec

Cost control should be a first-class resource.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Budget
metadata:
  name: research-team-budget
spec:
  scope:
    namespace: research

  limits:
    dailyUsd: 100
    monthlyUsd: 2000
    perExecutionUsd: 10
    tokenLimitDaily: 10000000

  actions:
    onSoftLimit: warn
    onHardLimit: suspend

  allocation:
    byAgent:
      research-agent: 0.5
      writer-agent: 0.3
      reviewer-agent: 0.2

status:
  spentTodayUsd: 42.12
  spentMonthUsd: 880.44
```

---
