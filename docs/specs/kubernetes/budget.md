# Budget

A `Budget` is a cost or usage cap that Joch enforces before model calls, tool calls, or executions exceed it. Budgets compose with [`Policy`](policy.md) so the same enforcement point handles cost rules and operational rules uniformly.

[Back to the catalog](index.md)

## Spec and status

```yaml
apiVersion: joch.dev/v1alpha1
kind: Budget
metadata:
  name: support-platform-monthly
  namespace: support-platform
spec:
  description: Monthly cost cap for the support-platform team.

  scope:
    teams:
      - support-platform

  limits:
    monthlyUsd: 5000
    dailyUsd: 250
    perRunUsd: 5
    tokenLimitDaily: 10000000

  enforcement:
    onSoftLimit: alert
    onHardLimit: deny
    softLimitPct: 0.8

  allocation:
    byAgent:
      support-triage: 0.5
      support-escalation: 0.3
      billing-agent: 0.2

  attribution:
    model: true
    tool: true
    approvals: true

status:
  phase: Ready
  monthSpentUsd: 1421.30
  daySpentUsd: 64.10
  recentRuns:
    overSoftLimit: 0
    overHardLimit: 0
```

## Where enforcement happens

```text
Model router       checks per-call cost against perRunUsd before dispatch
Tool gateway       attributes tool costs and checks per-run / per-day
Approval service   attributes approval costs (operator time) where applicable
Eval service       attributes eval cost to the budget that owns the eval
```

A budget breach is a `BudgetExceeded` event in the trace and may terminate the execution mid-flight if `onHardLimit: deny`.

## Attribution

Costs roll up by model, tool, agent, namespace, and team. `joch cost by-team` and `joch cost by-agent` use the same attribution Joch reports here.

[Back to the catalog](index.md)
