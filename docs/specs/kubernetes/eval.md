# Eval

An `Eval` is a versioned, scored evaluation of an agent against a dataset. Joch evals power release gates, regression detection, and cost / quality monitoring. Each eval is a first-class resource so its results are durable, comparable, and auditable.

[Back to the catalog](index.md)

## Spec and status

```yaml
apiVersion: joch.dev/v1alpha1
kind: Eval
metadata:
  name: support-triage-quality
  namespace: support-platform
spec:
  target:
    kind: Agent
    name: support-triage

  dataset:
    type: file
    uri: ./evals/support-cases.jsonl

  metrics:
    - name: task_success
      type: llm_judge
      judgeModelRef: { name: gpt-5-mini }
    - name: tone_consistency
      type: llm_judge
      judgeModelRef: { name: gpt-5-mini }
    - name: cost_per_case
      type: numeric
    - name: latency_p95_ms
      type: numeric
    - name: pii_leakage
      type: deterministic

  schedule:
    onDeployment: true
    nightly: true

  thresholds:
    task_success: 0.85
    tone_consistency: 0.9
    cost_per_case: "<0.40"
    latency_p95_ms: "<3000"
    pii_leakage: 0.0

  releaseGate:
    enabled: true
    blocksPromotionTo: prod

status:
  phase: Passed
  lastRunAt: "2026-05-10T02:00:00Z"
  scores:
    task_success: 0.91
    tone_consistency: 0.94
    cost_per_case: 0.32
    latency_p95_ms: 2750
    pii_leakage: 0.0
```

## Metric types

```text
llm_judge        a configured judge model rates the output on a rubric
deterministic    code-driven check (regex / classifier / schema)
numeric          measured value (cost, latency, token usage)
```

LLM-judge metrics carry the judge model in the result so the score is reproducible.

## Release gate

When `releaseGate.enabled` is true, a failing eval blocks `joch promote` of the target agent into `releaseGate.blocksPromotionTo`. The promotion failure includes the failing metric, the threshold, and the observed value.

## Schedule

```text
onDeployment    runs immediately after each apply or promote
nightly         runs at a fixed schedule
manual          operator-triggered only
```

Eval runs are recorded as [`Execution`](execution.md) records under the hood, with full traces and ABOM snapshots for reproducibility.

[Back to the catalog](index.md)
