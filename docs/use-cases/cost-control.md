# Cost Control

> Finance asks how much each team spent on AI agents last month, by model and by tool. The answer must be defensible, reproducible, and produced without instrumenting every SDK separately.

## Symptoms

- Provider bills are aggregate; the team that incurred the cost is invisible.
- Agents spike cost overnight and nobody knows why until the bill arrives.
- Setting a budget on one team has no effect because the limit lives in spreadsheets.
- A new agent ships with no cost expectation and runs unchecked.

## What Joch does

Cost is a first-class [observability primitive](../pillars/observability.md). The [Model Router](../architecture/model-router.md) and [Tool Gateway](../architecture/tool-gateway.md) record `costUsd` per call. The [`Budget`](../specs/kubernetes/budget.md) controller enforces caps before they are exceeded.

## Walkthrough

### 1. Set budgets

```yaml
apiVersion: joch.dev/v1alpha1
kind: Budget
metadata:
  name: support-platform-monthly
spec:
  scope: { teams: [support-platform] }
  limits: { monthlyUsd: 5000, dailyUsd: 250, perRunUsd: 5 }
  enforcement:
    onSoftLimit: alert
    onHardLimit: deny
    softLimitPct: 0.8
```

```bash
joch apply -f budgets/support-platform-monthly.yaml
```

### 2. View attribution

```bash
joch cost by-team --since 30d
joch cost by-agent --since 7d
joch top models --by cost --since 30d
joch top tools  --by cost --since 30d
```

Every line item joins through `agent`, `team`, `environment`, `model`, `tool`, and `time-window`.

### 3. Investigate spikes

```bash
joch executions ls --over-cost 4 --since 24h
joch describe execution exec-20260510-014
joch trace exec-20260510-014
```

The trace surfaces the model calls and tool calls that drove the cost, with token counts and per-call cost.

### 4. Respond with policy

```yaml
apiVersion: joch.dev/v1alpha1
kind: Policy
metadata: { name: research-cost-cap }
spec:
  appliesTo:
    agents: { selector: { matchLabels: { team: research } } }
  rules:
    - when: { cost.runUsd: ">3" }
      action: { deny: true, reason: research-cap-exceeded }
    - when: { model.provider: openai, model.tier: premium }
      require: { approval: human }
```

### 5. Optimize routes

```yaml
apiVersion: joch.dev/v1alpha1
kind: ModelRoute
metadata: { name: research-default }
spec:
  requirements: { toolCalling: true, contextWindowMin: 200000 }
  strategy:
    primary: anthropic:claude-haiku
    fallback:
      - openai:gpt-5-thinking
  constraints: { maxCostUsdPerExecution: 2 }
```

Switching the primary in one place reroutes every agent that uses the route. No agent code changes.

## Resources involved

- [`Budget`](../specs/kubernetes/budget.md)
- [`ModelRoute`](../specs/kubernetes/model-route.md), [`Model`](../specs/kubernetes/model.md)
- [`Policy`](../specs/kubernetes/policy.md)
- [`Trace`](../specs/kubernetes/trace.md)

## Outcome

Finance gets defensible cost reports. Engineering gets enforceable caps before bills arrive. Optimizing cost is a route or budget change, not a code change.
