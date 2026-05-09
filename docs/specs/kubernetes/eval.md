# Eval

Kubernetes-style YAML resource specification for Joch `Eval` resources.

[Back to Kubernetes specs](index.md)

## Eval spec

we need evals to prevent silent regressions.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Eval
metadata:
  name: research-agent-citation-eval
spec:
  target:
    kind: Agent
    name: research-agent

  dataset:
    type: file
    uri: ./evals/research-cases.jsonl

  metrics:
    - name: factuality
      type: llm_judge
    - name: citation_accuracy
      type: deterministic
    - name: task_success
      type: llm_judge
    - name: cost
      type: numeric
    - name: latency
      type: numeric

  schedule:
    onDeployment: true
    nightly: true

  thresholds:
    factuality: 0.9
    citation_accuracy: 0.95
    task_success: 0.85

status:
  phase: Passed
  lastRunAt: "2026-05-09T02:00:00Z"
  scores:
    factuality: 0.93
    citation_accuracy: 0.97
```

---
