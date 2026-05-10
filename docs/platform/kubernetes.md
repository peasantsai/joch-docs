# Kubernetes Role

Kubernetes should run a large part of the Joch platform, but raw Kubernetes should not be the product abstraction.

Kubernetes is excellent at managing containers and infrastructure resources:

```yaml
kind: Deployment
kind: Job
kind: Secret
kind: ConfigMap
kind: Service
kind: HorizontalPodAutoscaler
```

An agent platform needs higher-level resources:

```text
Agent
Model
Personality
Skill
Tool
MCPServer
Memory
RAG
Execution
Plan
ToolCall
Approval
Conversation
StateCheckpoint
Trace
Budget
Policy
Guardrail
```

Kubernetes does not natively answer agent-specific questions:

```text
Which model can run this agent?
Can this agent call this MCP tool?
Does this tool call require approval?
Which memory store is writable?
What happens if the agent switches from OpenAI to Claude?
Did this run exceed its token budget?
Was this output grounded in the RAG sources?
Should this conversation checkpoint be compacted?
Did this execution produce a side-effectful tool call?
```

Kubernetes can store agent resources through CRDs, but Joch still needs to define and enforce the agent semantics.

## What Kubernetes Provides

Kubernetes already provides the infrastructure machinery Joch should reuse:

```text
Resource API
Declarative apply
Watch API
Reconciliation model
Namespaces
RBAC
Secrets
Events
Labels/selectors
Status subresources
Finalizers
Owner references
Rollbacks
Health probes
Scheduling
Autoscaling
```

This gives Joch battle-tested scheduling, scaling, secret handling, networking, RBAC, health checks, and rollout machinery without forcing users to think in Kubernetes primitives.

## When Kubernetes Is Enough

Plain Kubernetes is enough if the goal is only to:

```text
Run agent containers
Scale them
Restart them if they crash
Expose them over HTTP
Mount secrets
```

In that simple model:

```text
Deployment = agent runtime
ConfigMap = prompt/personality
Secret = API keys
Job = one-off execution
PVC = memory
Service = API endpoint
```

That works for a basic runtime platform.

## When Joch Is Necessary

Joch becomes necessary when the platform needs:

```text
Cross-vendor model routing
MCP tool governance
Agent memory lifecycle
RAG indexing
Tool-call audit trails
Human approvals
Conversation portability
Provider failover
Cost budgets
Agent-specific evals
Personality-as-code
Multi-agent handoffs
Execution traces
Policy enforcement before tool use
```

Kubernetes does not provide those semantics out of the box. It provides the machinery Joch can use to build them.
