# ModelRoute

A `ModelRoute` is a capability-aware, cost-aware, region-aware fallback policy across providers. Agents reference a `ModelRoute` instead of a single model. The [model router](../../architecture/model-router.md) consumes the route at every model call.

[Back to the catalog](index.md)

## Spec

```yaml
apiVersion: joch.dev/v1alpha1
kind: ModelRoute
metadata:
  name: research-default
  namespace: research
spec:
  description: >
    Default model route for research and analysis agents.
    Prefers OpenAI gpt-5-thinking, falls back to Claude Sonnet, then Gemini Pro.

  requirements:
    toolCalling: true
    contextWindowMin: 200000
    structuredOutput: true
    streaming: true

  strategy:
    primary: openai:gpt-5-thinking
    fallback:
      - anthropic:claude-sonnet
      - google:gemini-pro
    failover:
      onError: true
      onBudgetExceeded: true
      onCapabilityMismatch: true
      onRegionMismatch: true
      onHealthDegraded: true

  constraints:
    maxCostUsdPerExecution: 5
    maxCostUsdPerCall: 1
    dataResidency: EU

  observability:
    explainOnFallback: true

status:
  phase: Ready
  primaryHealth: Healthy
  fallbackHealth:
    anthropic:claude-sonnet: Healthy
    google:gemini-pro: Healthy
  fallbackRate7d: 0.012
```

## Resolution

At call time the router:

1. Selects the highest-priority candidate that satisfies `requirements`.
2. Enforces `constraints` against the running execution (cost so far, region of origin).
3. Asks the policy engine if the candidate is permitted in this context.
4. Calls the provider via its adapter.
5. On `failover` triggers, tries the next candidate in `strategy.fallback`.

Every fall-forward is recorded as a `ProviderSwitched` event in the trace, with the reason. `joch route exec-123 --explain` reconstructs the decision after the fact.

## Why a route, not a single model

A `Model` reference would couple agents to a single provider. The route abstracts the provider choice so:

- Provider outages do not crash the fleet.
- Cost spikes can shift load to a cheaper backend.
- Capability mismatches at one provider do not propagate to the agent.
- Region or residency rules can be enforced uniformly.
- Migration between providers is a one-line route edit, applied across every agent that references it.

## Constraints

`constraints` are evaluated at the running-execution scope. A `maxCostUsdPerExecution` is checked against the `Execution`'s cumulative cost; a hit denies the call rather than silently truncating.

`dataResidency` matches against the `Model`'s `routing.regions`. A request with `dataResidency: EU` will only choose models whose `regions` includes `eu`.

[Back to the catalog](index.md)
