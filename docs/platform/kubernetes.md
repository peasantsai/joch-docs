# Kubernetes Strategy

we **should use Kubernetes** for a lot of it.

But we probably should **not expose raw Kubernetes as the product abstraction**.

The right answer is likely:

```text
Use Kubernetes as the infrastructure substrate.
Build joch as the agent control plane above it.
```

Not:

```text
Rebuild Kubernetes from scratch.
```

And not:

```text
Force users to model agents directly as Pods, Deployments, ConfigMaps, and Jobs.
```

---

## Why Kubernetes is not enough by itself

Kubernetes is excellent at managing **containers**.

It knows how to run things like:

```yaml
kind: Deployment
kind: Job
kind: Secret
kind: ConfigMap
kind: Service
kind: HorizontalPodAutoscaler
```

But an agent platform needs to manage higher-level resources:

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

Kubernetes does not natively understand:

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

we can make Kubernetes store those things using CRDs, but the **agent semantics** still need to be built.

---

## Best architecture: Kubernetes underneath, joch above

I would design it like this:

```text
joch CLI
  ↓
joch API / controller layer
  ↓
Kubernetes CRDs + controllers
  ↓
Pods, Jobs, Deployments, Secrets, Services
```

So `joch` resources become Kubernetes custom resources:

```yaml
apiVersion: joch.dev/v1alpha1
kind: Agent
metadata:
  name: research-agent
spec:
  modelRef:
    name: gpt-5-thinking
  personalityRef:
    name: pragmatic-researcher
  tools:
    - web.search
    - github.create_issue
```

Then our controller reconciles that into lower-level Kubernetes objects:

```text
AgentDeployment
  ↓
Kubernetes Deployment
Kubernetes Secret
Kubernetes ServiceAccount
Kubernetes NetworkPolicy
Kubernetes ConfigMap
```

For one-off runs:

```text
Execution
  ↓
Kubernetes Job
```

For long-running agents:

```text
Deployment
  ↓
Kubernetes Deployment / StatefulSet
```

For scheduled agents:

```text
Schedule
  ↓
Kubernetes CronJob
```

This gives we Kubernetes’ battle-tested scheduling, scaling, secrets, networking, RBAC, health checks, and rollout machinery without making the user think in Kubernetes primitives.

---

## What `joch` adds on top

Kubernetes answers:

```text
How do I run this workload?
Where should it be scheduled?
Is the container alive?
How many replicas are ready?
Can this pod access this secret?
```

`joch` answers:

```text
What is this agent?
What model does it use?
What tools can it call?
What memory can it read?
What policy governs it?
What plan is it executing?
What did it do?
What did it cost?
Can it switch providers?
Can I replay or audit it?
```

That is the product layer.

---

## Kubernetes CRDs are probably the right implementation

Instead of building a separate API server and database first, we could start with Kubernetes CRDs:

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

Then build controllers:

```text
AgentController
ExecutionController
MCPDiscoveryController
MemoryController
RAGController
PolicyController
DeploymentController
```

This is elegant because Kubernetes already gives you:

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

So yes: use Kubernetes as much as possible.

---

## But do not make Kubernetes mandatory for everyone

There is one big product problem: not every user wants a cluster.

our audience may include:

```text
Solo developers
Local-first hackers
Small teams
CI/CD users
Enterprise platform teams
```

So I would support two modes.

### 1. Local mode

```bash
joch up
joch apply -f agent.yaml
joch run research-agent
```

Under the hood:

```text
SQLite/Postgres
Local worker process
Local MCP servers
Docker optional
```

This is for development.

### 2. Cluster mode

```bash
joch install --cluster
joch apply -f agent.yaml
joch deploy research-agent --env prod
```

Under the hood:

```text
Kubernetes CRDs
Controllers
Jobs
Deployments
Secrets
Services
NetworkPolicies
OpenTelemetry
```

This is for production.

That mirrors a pattern developers already understand:

```text
Docker Compose locally
Kubernetes in production
```

For `joch`:

```text
joch local for dev
joch on Kubernetes for prod
```

---

## The key design choice

Do not clone Kubernetes.

Instead, build a **domain-specific control plane** that can use Kubernetes as a backend.

The abstraction stack should be:

```text
User-facing:
  Agent, Skill, Model, Memory, Execution, ToolCall, Policy

Control-plane:
  joch controllers and compilers

Infrastructure-facing:
  Kubernetes Jobs, Deployments, Secrets, Services, ConfigMaps
```

The user should say:

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

---

## When plain Kubernetes is enough

If our goal is only:

```text
Run agent containers
Scale them
Restart them if they crash
Expose them over HTTP
Mount secrets
```

then Kubernetes alone is enough.

we could define:

```text
Deployment = agent runtime
ConfigMap = prompt/personality
Secret = API keys
Job = one-off execution
PVC = memory
Service = API endpoint
```

That works for a simple platform.

---

## When `joch` becomes necessary

we need `joch` once we care about:

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

Kubernetes will not give we those semantics out of the box.

It gives we the machinery to build them.

---

## Concrete example

A Kubernetes-native implementation could look like this:

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

our `ExecutionController` creates:

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

The runtime worker reads the `Execution`, loads the compiled `Agent`, talks to the model router, calls MCP tools through the tool gateway, writes traces, then updates:

```yaml
status:
  phase: Succeeded
  costUsd: 1.42
  toolCalls: 17
  artifactRef:
    name: report-123
```

Kubernetes runs the container.

`joch` understands the agent.

---

## Recommended product strategy

I would do this:

```text
1. Define joch resource specs independent of Kubernetes.
2. Implement a local backend first for fast iteration.
3. Implement a Kubernetes backend using CRDs/controllers.
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

That gives we portability.

---

## Final answer

we are right to challenge the idea of a tailored clone.

The correct design is:
