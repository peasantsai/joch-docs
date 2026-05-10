# Product Strategy

Joch should not clone Kubernetes. It should define an agent-native control plane that can use Kubernetes as a backend.

## Recommended Path

The product strategy should be:

```text
1. Define Joch resource specs independent of Kubernetes.
2. Implement a local backend first for fast iteration.
3. Implement a Kubernetes backend using CRDs and controllers.
4. Make the CLI backend-agnostic.
```

Example:

```bash
joch context use local
joch apply -f agent.yaml

joch context use prod-k8s
joch apply -f agent.yaml
```

Same spec, different backend.

## Why This Works

This gives Joch:

```text
Fast local development
Production-grade Kubernetes operations
One resource model across environments
Room for managed SaaS later
Agent-native UX instead of raw infrastructure UX
```

It also keeps the CLI focused:

```bash
joch run research-agent --task "Analyze this market"
```

Not:

```bash
kubectl create job research-agent-run-123 \
  --image=agent-runtime \
  --env=MODEL=gpt-5 \
  --env=PERSONALITY_CONFIGMAP=...
```

## Final Position

The right design is:

```text
Kubernetes underneath.
Joch above it.
Agent resources at the product boundary.
Infrastructure resources behind adapters and controllers.
```

That gives Joch portability without giving up Kubernetes' operational strengths.
