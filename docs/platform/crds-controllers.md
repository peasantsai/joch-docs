# CRDs and Controllers

Kubernetes CRDs are the right production implementation path once the Joch resource model is stable.

Instead of starting with a separate API server and database for every environment, Joch can define custom resources:

```text
Agent.joch.dev
Model.joch.dev
Skill.joch.dev
Tool.joch.dev
MCPServer.joch.dev
Memory.joch.dev
RAG.joch.dev
Execution.joch.dev
Policy.joch.dev
Guardrail.joch.dev
Approval.joch.dev
Trace.joch.dev
```

Then Joch can add controllers:

```text
AgentController
ExecutionController
MCPDiscoveryController
MemoryController
RAGController
PolicyController
DeploymentController
```

## Example Execution Flow

A Kubernetes-native execution resource could look like this:

```yaml
apiVersion: joch.dev/v1alpha1
kind: Execution
metadata:
  name: exec-123
spec:
  agentRef:
    name: research-agent
  input:
    content: "Research the agent orchestration market."
```

The `ExecutionController` creates a worker job:

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: exec-123-worker
spec:
  template:
    spec:
      containers:
        - name: agent-runtime
          image: ghcr.io/acme/agent-runtime:1.0
          env:
            - name: EXECUTION_ID
              value: exec-123
```

The runtime worker reads the `Execution`, loads the compiled `Agent`, talks to the model router, calls MCP tools through the tool gateway, writes traces, and updates status:

```yaml
status:
  phase: Succeeded
  costUsd: 1.42
  toolCalls: 17
  artifactRef:
    name: report-123
```

Kubernetes runs the container. Joch understands the agent.

## Controller Boundary

Controllers should reconcile Joch resources into Kubernetes resources, but they should not leak Kubernetes primitives into the user experience. The user should apply agent specs; the operator should own jobs, services, service accounts, policies, secrets, and rollout details.
