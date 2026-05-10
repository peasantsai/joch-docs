# Release Gates

> A new version of a customer-facing agent must not promote into production unless a quality eval passes, an ABOM diff shows no high-risk components, and a release manager approves.

## Symptoms

- Prompt or tool changes regress quality and customers notice before engineering does.
- A prod deploy adds a previously-unpinned MCP server that no one reviewed.
- Rollbacks are manual and error-prone after a regression slips through.
- Audit cannot answer "who promoted this version, when, after which checks."

## What Joch does

Joch's [Release Management pillar](../pillars/release-management.md) wires [`Eval`](../specs/kubernetes/eval.md), [`ABOM`](../specs/kubernetes/abom.md), and [`Approval`](../specs/kubernetes/approval.md) into `joch promote` as enforceable gates.

## Walkthrough

### 1. Define the eval

```yaml
apiVersion: joch.dev/v1alpha1
kind: Eval
metadata: { name: support-triage-quality }
spec:
  target: { kind: Agent, name: support-triage }
  dataset: { type: file, uri: ./evals/support-cases.jsonl }
  metrics:
    - { name: task_success,    type: llm_judge }
    - { name: tone_consistency,type: llm_judge }
    - { name: cost_per_case,   type: numeric }
    - { name: latency_p95_ms,  type: numeric }
    - { name: pii_leakage,     type: deterministic }
  schedule: { onDeployment: true, nightly: true }
  thresholds:
    task_success:    0.85
    tone_consistency:0.9
    cost_per_case:   "<0.40"
    latency_p95_ms:  "<3000"
    pii_leakage:     0.0
  releaseGate: { enabled: true, blocksPromotionTo: prod }
```

### 2. Wire the deployment gate

```yaml
apiVersion: joch.dev/v1alpha1
kind: Deployment
metadata: { name: support-triage-prod }
spec:
  agentRef: { name: support-triage }
  environmentRef: { name: prod }
  promotion:
    requiresEval: support-triage-quality
    requiresApproval: release-manager
```

### 3. Run the promotion

```bash
joch validate -f .
joch diff agent support-triage --from prod --to staging
joch eval run support-triage-quality --target staging
joch promote support-triage --from staging --to prod
```

The promote command fails if the eval is below threshold, if the ABOM diff flags a high-risk component, or if the release manager has not approved.

### 4. Roll back

```bash
joch rollback agent support-triage --to-version 7
```

The rollback re-applies the prior agent record, including its referenced model, tool, MCP server, and policy versions. The rollback is logged as a release event with the reason.

### 5. Audit

```bash
joch get releases --agent support-triage
joch describe release rel-2026-05-10-001
```

Each release record carries the agent diff, eval results, ABOM diff, approver, environment, runtime context, and timestamp.

## Resources involved

- [`Eval`](../specs/kubernetes/eval.md), [`ABOM`](../specs/kubernetes/abom.md), [`Approval`](../specs/kubernetes/approval.md)
- [`Deployment`](../specs/kubernetes/deployment.md), [`Environment`](../specs/kubernetes/environment.md)
- [`Policy`](../specs/kubernetes/policy.md)

## Outcome

Production agents stop regressing because the gate is mandatory. Audit gets a defensible release history. Engineering gets one-command rollback and a faster recovery loop.
