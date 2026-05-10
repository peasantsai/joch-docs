# Kubernetes Role

Kubernetes is an excellent substrate for Joch. It is not the right product surface for AI agent fleets.

## What Kubernetes provides

Kubernetes natively offers:

```text
Resource API
Declarative apply
Watch API
Reconciliation model
Namespaces
RBAC
Secrets
Events
Labels and selectors
Status subresources
Finalizers
Owner references
Rollbacks
Health probes
Scheduling
Autoscaling
NetworkPolicies
```

Joch reuses every one of these where it runs against Kubernetes. There is no value in re-implementing them.

## What Kubernetes does *not* answer

Kubernetes models containers, not agents. It does not natively answer:

```text
Which model can run this agent?
Can this agent call this MCP tool?
Does this tool call require approval?
Which memory store is writable?
What happens if the agent switches from OpenAI to Claude?
Did this run exceed its token budget?
Was this output grounded in the RAG sources?
Should this conversation checkpoint be compacted?
Did this execution produce a side-effecting tool call?
```

Those are agent semantics. Kubernetes can store agent records as CRDs, but Joch defines and enforces the semantics.

## The split

```text
Kubernetes runs the workloads.
Joch understands the agents.
```

A Joch [`Deployment`](../specs/kubernetes/deployment.md) reconciles into a Kubernetes Deployment via the [Kubernetes runtime adapter](../architecture/service-architecture.md). A Joch [`Execution`](../specs/kubernetes/execution.md) reconciles into a Kubernetes Job. A Joch [`Secret`](../specs/kubernetes/secret.md) references a Kubernetes Secret.

The reverse is not true: a Kubernetes Deployment knows nothing about an `Agent`, a `ModelRoute`, an `MCPServer`, an `AgBOM`, or a `Policy`. That is the layer Joch adds.

## When raw Kubernetes is enough

Plain Kubernetes is enough if all you need is:

```text
Run an agent container
Scale it
Restart it on crash
Expose it over HTTP
Mount secrets
```

A small team with one agent can ship that with `Deployment + ConfigMap + Secret + Job + PVC + Service`.

## When Joch is necessary

Joch becomes necessary when you need:

```text
Cross-vendor model routing
MCP tool governance and version pinning
Agent memory lifecycle
RAG indexing
Tool-call audit trails and idempotency
Human approvals
Conversation portability
Provider failover
Cost budgets and per-team attribution
Agent-specific evals and release gates
Multi-agent handoffs (A2A)
Execution traces with OpenTelemetry / OCSF export
Policy enforcement before tool / model / memory use
Agent Bill of Materials
```

Kubernetes does not answer any of those out of the box. Joch builds them on top.

## Why Joch on Kubernetes is the production sweet spot

- Joch reuses Kubernetes' battle-tested scheduling, RBAC, and rollout machinery.
- Operators can use `kubectl` for low-level introspection and `joch` for agent-level operations.
- Helm and Operators are familiar to the platform teams that buy Joch.
- NetworkPolicies make the [trust zones](../architecture/trust-and-security-model.md) enforceable at the cluster network layer.

For dev and small deployments, [Local mode and Docker Compose](deployment-modes.md) provide the same UX without Kubernetes.
