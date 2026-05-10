# Product Strategy

The platform-level product strategy is small and disciplined: define resources once, run them anywhere, and grow with the SDK ecosystem rather than fight it.

## The path

1. **Define Joch resources independent of any single backend.** Records are framework-agnostic and runtime-agnostic.
2. **Build the local backend first** for fast iteration, demos, and CI.
3. **Build the Kubernetes backend** with CRDs, an operator, and a Helm chart for production.
4. **Keep the CLI backend-agnostic.** Same specs, same commands, different context.

```bash
joch context use local
joch apply -f agent.yaml

joch context use prod-k8s
joch apply -f agent.yaml
```

Same spec, different runtime adapter.

## Why this works

- Fast local development without infrastructure overhead.
- Production-grade Kubernetes operations without a separate model.
- One resource catalog across environments.
- Room for managed SaaS without forking the model.
- Agent-native UX instead of raw infrastructure UX.

The CLI stays focused on the agent intent:

```bash
joch run support-triage --task "Triage the queue"
```

— not on the infrastructure plumbing:

```bash
kubectl create job support-triage-run-123 \
  --image=joch-worker \
  --env=JOCH_EXECUTION_ID=...
```

## Final position

```text
Kubernetes (or Docker, or local) underneath.
Joch above it.
Agent resources at the product boundary.
Infrastructure resources behind adapters and controllers.
```

That gives Joch portability without giving up Kubernetes' operational strengths.
