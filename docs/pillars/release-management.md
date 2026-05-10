# Release Management

> Version, eval, diff, promote, roll back, and audit agents like production software, with quality gates that block regressions before they reach customers.

Release management turns ad-hoc prompt iteration into production engineering. The same discipline that keeps web services from regressing — versioning, evals, gated promotion, rollback — applies to agents, but the failure modes are different and the tooling has to match.

## Versioning

Every applied resource is a version. Joch records the resource version, the active policy versions at the time, and the framework adapter version, so the runtime configuration is fully reproducible.

```bash
joch get agents support-triage --history
joch describe agent support-triage --version 8
joch diff agent support-triage --from 7 --to 8
```

The diff is structural (model, tools, MCP servers, policies, budgets, framework adapter) and includes referenced resources, so a policy change that affects the agent is visible in the diff even if the agent record itself did not change.

## Evals

A Joch [`Eval`](../specs/kubernetes/eval.md) is a versioned, scored evaluation against a dataset:

```yaml
apiVersion: joch.dev/v1alpha1
kind: Eval
metadata:
  name: support-triage-quality
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
    - name: tone_consistency
      type: llm_judge
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
```

Evals run on demand, on a schedule, or as a release gate. Their results are durable and feed [observability](observability.md).

## Diff and promote

Joch encourages an explicit promotion ladder:

```text
local → dev → staging → prod
```

Each step is a `joch promote` with optional gates. A typical CI pipeline:

```bash
joch validate -f .
joch diff agent support-triage --from prod --to staging
joch eval run support-triage-quality --target staging
joch promote support-triage --from staging --to prod \
  --require eval:support-triage-quality \
  --require approval:release-manager
```

A promotion that fails its gate produces a clear failure and an audit record.

## Release gates

A release gate is a check that runs before a promotion is allowed:

```text
Eval gate           required eval must pass thresholds
Approval gate       a designated team must approve
Policy gate         the new spec must pass all policy validations
ABOM gate           ABOM diff must show no high-risk new components
Budget gate         projected cost must stay below budget
Latency gate        latency regression must stay below threshold
```

Release gates compose. A `prod` promotion can require any combination, with each contributing a verifiable record to the audit trail.

## Rollback

Rollback is one command:

```bash
joch rollback agent support-triage --to-version 7
```

The rollback re-applies the prior agent record, including its referenced model, tool, MCP server, and policy versions. Joch records the rollback as a release event with the reason.

## CI/CD integration

Joch fits into existing CI:

```yaml
# .github/workflows/agents.yml (example)
- name: Validate Joch resources
  run: joch validate -f ./agents

- name: Diff against prod
  run: joch diff -f ./agents --against context:prod

- name: Run evals
  run: joch eval run -f ./evals --target staging

- name: Promote
  if: github.ref == 'refs/heads/main'
  run: joch promote -f ./agents --from staging --to prod
```

The same commands work locally for fast iteration and in CI for governed promotion.

## What a Joch release record contains

Every promotion creates a release record with:

```text
agentRef             which agent
fromVersion          previous version
toVersion            new version
diff                 structural diff (resources + referenced policies)
evalResults          which evals ran, which thresholds passed
approvals            who approved, when
abomDiff             new components, removed components, changed versions
runtimeContext       which environment / cluster / cloud
operatorIdentity     who or which automation triggered the promotion
timestamp            UTC
rollbackPath         the rollback target if needed
```

These records are the auditable history of every change to a production agent.

## Acceptance criteria

A team operating Joch's Release Management pillar can:

- prove that prod agent X is at version Y, applied at time T, by operator O, with policies P at versions V,
- block a release that drops `task_success` below threshold, with an actionable failure message,
- roll back a regression in one command, with the rollback recorded in the audit log,
- diff two agent versions and see the structural change plus the referenced policy and ABOM deltas.
