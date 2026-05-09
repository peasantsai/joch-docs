# Deployment

Kubernetes-style YAML resource specification for Joch `Deployment` resources.

[Back to Kubernetes specs](index.md)

## Deployment spec

A `Deployment` runs agents as a managed fleet.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Deployment
metadata:
  name: research-agent-prod
spec:
  agentRef:
    name: research-agent

  replicas: 3

  strategy:
    type: rolling
    maxUnavailable: 1
    maxSurge: 1

  runtime:
    type: container
    image: ghcr.io/acme/agent-runtime:1.4.0

  environmentRef:
    name: prod-eu

  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 20
    metrics:
      - type: queue_depth
        target: 20
      - type: latency_p95
        targetMs: 5000

  scheduling:
    region: eu
    nodeSelector:
      workload: agent-runtime

  rollout:
    canary:
      enabled: true
      percentage: 10
      duration: 30m

  healthChecks:
    readiness:
      type: synthetic_prompt
      prompt: "Reply with READY in JSON."
    liveness:
      type: heartbeat
      interval: 30s

status:
  phase: Available
  readyReplicas: 3
  currentRevision: 12
```

This is the resource that makes `joch` a real fleet manager, not just a local CLI.

---
