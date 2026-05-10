# CRDs and Controllers

When Joch runs on Kubernetes, its resources can be exposed as native Custom Resource Definitions. Operators reconcile those CRDs into runtime artifacts (Deployments, Jobs, ConfigMaps, Secrets) without leaking Kubernetes primitives into the user experience.

## CRD groups

```text
joch.dev
  Agent
  FrameworkAdapter
  Policy

runtime.joch.dev
  Execution
  ToolCall
  Handoff
  Approval
  Conversation
  StateCheckpoint

model.joch.dev
  Model
  ModelRoute

tools.joch.dev
  Tool
  MCPServer

memory.joch.dev
  Memory
  RAG
  KnowledgeSource

ops.joch.dev
  Deployment
  Environment
  Team
  Budget
  Trace
  Eval
  Artifact
  AgBOM
  Secret
```

## Controllers

Each kind has a controller. Controllers reconcile the desired spec into runtime artifacts and update `status`.

```text
AgentController              compiles Agent + FrameworkAdapter into a CompiledAgentManifest
ExecutionController          materializes a Job + worker, watches for completion
DeploymentController         maintains the desired replica count and rollout strategy
PolicyController             distributes compiled policies to the policy engine
MCPDiscoveryController       refreshes MCPServer.status.discoveredTools
AgBOMController              regenerates AgBOM on dependency changes
EvalController               runs Evals on schedule and on promotion
ApprovalController           routes approval requests; records decisions
BudgetController             tracks cost and enforces caps
```

Controllers run inside `joch-controller-manager`, deployed as a single binary or split per group.

## Example: Execution → Job

A Kubernetes-native Execution:

```yaml
apiVersion: runtime.joch.dev/v1alpha1
kind: Execution
metadata:
  name: exec-20260510-001
  namespace: support-platform
spec:
  agentRef: { name: support-triage }
  input:
    content:
      - kind: data
        data: { ticketId: T-77123 }
```

The `ExecutionController` produces a worker Job:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: exec-20260510-001
  namespace: support-platform
spec:
  template:
    spec:
      serviceAccountName: joch-worker
      containers:
        - name: agent-runtime
          image: ghcr.io/peasantsai/joch-worker:1.0.0
          env:
            - name: JOCH_EXECUTION_ID
              value: exec-20260510-001
            - name: JOCH_GATEWAY_URL
              value: http://joch-gateway
            - name: JOCH_ROUTER_URL
              value: http://joch-router
            - name: JOCH_TRACE_URL
              value: http://joch-trace
```

The worker runs the configured framework adapter, talks to the data plane, and updates Execution status:

```yaml
status:
  phase: Succeeded
  costUsd: 0.62
  toolCalls: 4
  artifactRefs:
    - artifact://exec/exec-20260510-001/draft-reply.md
```

## Controller boundary

Controllers reconcile Joch resources into Kubernetes resources, but they do not leak Kubernetes primitives into the user experience. Users apply agent specs; the operator owns Jobs, Services, ServiceAccounts, NetworkPolicies, and rollout details.

Operators see one model:

```bash
joch get agents
joch get executions
joch describe agent support-triage
```

— or, for low-level inspection, the equivalent `kubectl` commands. Either way, they never assemble a Job by hand.
