# Deployment

A `Deployment` describes where and how many instances of an agent run, with what scaling, in what environment. Joch's controller manager reconciles `Deployment` records into runtime artifacts via the active [runtime adapter](../../architecture/service-architecture.md) (local processes, Docker containers, Kubernetes Deployments / StatefulSets / Jobs).

[Back to the catalog](index.md)

## Spec and status

```yaml
apiVersion: joch.dev/v1alpha1
kind: Deployment
metadata:
  name: support-triage-prod
  namespace: support-platform
spec:
  agentRef: { name: support-triage }
  environmentRef: { name: prod }

  replicas: 3
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 12
    metrics:
      - type: queueDepth
        target: 50
      - type: latencyP95
        targetMs: 5000

  rollout:
    strategy: rolling
    maxSurge: 1
    maxUnavailable: 0
    canary:
      enabled: false
      percentage: 10
      duration: 30m

  runtime:
    adapter: kubernetes
    workerImage: ghcr.io/peasantsai/joch-worker:1.0.0

  scheduling:
    region: eu
    nodeSelector:
      workload: agent-runtime

  healthChecks:
    readiness:
      type: synthetic_prompt
      prompt: "Reply with READY in JSON."
    liveness:
      type: heartbeat
      interval: 30s

  promotion:
    requiresEval: support-triage-quality
    requiresApproval: release-manager

status:
  phase: Available
  replicasReady: 3
  rolloutStatus: stable
  currentAgentVersion: 14
```

## Strategy

```text
rolling      progressive replacement, default
recreate     teardown then create
canary       % traffic shift via routing
shadow       new version receives mirrored traffic, results compared offline
```

Strategy choice depends on the runtime adapter and the agent's risk profile. Side-effecting agents typically use `rolling` with `maxUnavailable: 0`.

## Promotion gates

`promotion.requiresEval` and `promotion.requiresApproval` make this `Deployment` impossible to apply without the named release gates passing. This is the link between [Eval](eval.md), [Approval](approval.md), and the runtime topology.

[Back to the catalog](index.md)
