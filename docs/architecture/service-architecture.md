# Service Architecture

> Do not rebuild Kubernetes.
> Use Kubernetes for infrastructure orchestration.
> Build `joch` for agent semantics.

Kubernetes should manage:

```text
Pods
Jobs
Deployments
Secrets
Services
Scheduling
Scaling
Health
Networking
```

`joch` should manage:

```text
Agents
Models
Skills
Personalities
Memories
RAG
MCP tools
Executions
Plans
Approvals
Traces
Budgets
Policies
Provider migration
```

So the product is not “Kubernetes for agents” in the sense of replacing Kubernetes.

It is better framed as:

> `joch` is a Kubernetes-native control plane for AI agents, with a local mode for developers.
Yes. That is probably the strongest architecture:

> Build `joch` as a set of portable services with clean APIs, then provide adapters so the same platform can run on Docker, Kubernetes, local dev, cloud VMs, or managed SaaS.

So instead of:

```text
joch = Kubernetes clone
```

or:

```text
joch = Kubernetes-only CRDs
```

make it:

```text
joch = agent control plane + portable service mesh + deployment adapters
```

---

# 1. High-level shape

```text
                    ┌────────────────────┐
                    │     joch CLI    │
                    └─────────┬──────────┘
                              │
                              ▼
                    ┌────────────────────┐
                    │  joch API       │
                    │  Control Plane      │
                    └─────────┬──────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        ▼                     ▼                     ▼
┌────────────────┐   ┌────────────────┐   ┌────────────────┐
│ Runtime Layer  │   │ Tool/MCP Layer │   │ Memory/RAG     │
│ Execution      │   │ Gateway        │   │ Layer          │
└────────────────┘   └────────────────┘   └────────────────┘
        │                     │                     │
        └─────────────────────┼─────────────────────┘
                              ▼
                    ┌────────────────────┐
                    │ Deployment Adapter │
                    └─────────┬──────────┘
                              │
       ┌──────────────────────┼──────────────────────┐
       ▼                      ▼                      ▼
   Docker              Kubernetes              Local Process
 Compose             CRDs/Helm/Operator          SQLite/Files
```

The key design is:

```text
Agent semantics are owned by joch.
Infrastructure execution is delegated to adapters.
```

---

# 2. Core product layers

I would design it in four layers:

```text
Layer 1: Agent API layer
Layer 2: Control-plane services
Layer 3: Runtime/data-plane services
Layer 4: Deployment adapters
```

## Layer 1: Agent API layer

This is what users and systems interact with.

```text
joch CLI
REST API
gRPC API
SDKs
Terraform provider
Kubernetes CRDs
GitHub Actions
CI/CD integrations
```

## Layer 2: Control-plane services

These own desired state and governance.

```text
API Server
Spec Registry
Admission Engine
Policy Engine
Controller Manager
Compiler
Scheduler
Approval Service
Budget Service
```

## Layer 3: Runtime/data-plane services

These actually run agent workloads.

```text
Execution Worker
Model Router
Tool Gateway
MCP Gateway
Memory Service
RAG Service
Artifact Service
Trace Service
Eval Worker
```

## Layer 4: Deployment adapters

These let the platform run anywhere.

```text
Local adapter
Docker adapter
Docker Compose adapter
Kubernetes adapter
Nomad adapter
Serverless adapter
Cloud VM adapter
SaaS adapter
```

---

# 3. Service map

A full install could look like this:

```text
joch-apiserver
joch-controller-manager
joch-scheduler
joch-compiler
joch-runtime-worker
joch-model-router
joch-tool-gateway
joch-mcp-registry
joch-memory-service
joch-rag-service
joch-artifact-service
joch-policy-engine
joch-approval-service
joch-observability
joch-eval-service
joch-secret-broker
joch-web-ui
```

But for v1, collapse it into fewer binaries:

```text
joch-server
  API server
  spec registry
  admission
  policy
  controller manager
  compiler

joch-worker
  execution engine
  model router client
  prompt compiler
  execution loops

joch-gateway
  tool gateway
  MCP gateway
  secret broker

joch-memory
  memory service
  RAG service

joch-observability
  traces
  logs
  metrics
```

This avoids over-microservicing too early.

---

# 4. Portable service contract

Each service should be usable through stable contracts.

For example:

```text
joch-apiserver
  REST/gRPC API

joch-worker
  Pulls Execution jobs from API/server queue

joch-tool-gateway
  Exposes ToolCall API

joch-memory
  Exposes Memory API

joch-rag
  Exposes Retrieval API

joch-model-router
  Exposes Model Inference API

joch-observability
  Accepts OpenTelemetry traces/events
```

The services should not assume Kubernetes exists.

They should only assume:

```text
There is a resource store.
There is a queue/event bus.
There is object storage.
There is a secrets provider.
There is a runtime adapter.
```

---

# 5. Deployment modes

## Mode A: Local dev

```bash
joch up
joch apply -f agent.yaml
joch run research-agent
```

Under the hood:

```text
SQLite or local Postgres
Local filesystem artifacts
Local worker process
Local MCP stdio/http servers
No Kubernetes required
```

Architecture:

```text
joch CLI
  ↓
joch-server
  ↓
local worker subprocess
  ↓
local sqlite / local files / local MCP
```

Good for:

```text
Solo developers
Testing specs
Building agents locally
Fast iteration
```

---

## Mode B: Docker Compose

```bash
joch init docker
docker compose up
joch context use docker
joch apply -f .
```

Services:

```yaml
services:
  joch-server:
    image: ghcr.io/acme/joch-server:latest
    ports:
      - "8080:8080"

  joch-worker:
    image: ghcr.io/acme/joch-worker:latest
    depends_on:
      - joch-server

  joch-gateway:
    image: ghcr.io/acme/joch-gateway:latest

  postgres:
    image: postgres:16

  redis:
    image: redis:7

  minio:
    image: minio/minio

  otel-collector:
    image: otel/opentelemetry-collector
```

Good for:

```text
Team dev environments
Self-hosted small deployments
CI smoke tests
Demos
```

---

## Mode C: Kubernetes

```bash
joch install kubernetes
joch apply -f .
```

Installed components:

```text
CRDs
Controller Manager
API Server
Runtime Workers
Tool Gateway
Memory/RAG Services
Model Router
Observability
Helm chart
Operator
```

Kubernetes resources:

```text
Deployments
Jobs
Services
Secrets
ConfigMaps
NetworkPolicies
HorizontalPodAutoscalers
ServiceAccounts
CRDs
```

Good for:

```text
Production
Enterprise
Multi-tenant workloads
Scaling
Governance
```

---

## Mode D: Managed SaaS control plane + customer runtime

This is likely the best enterprise model.

```text
joch cloud control plane
  manages specs, UI, policies, traces

customer runtime plane
  runs workers, tools, memory, private data
```

Architecture:

```text
              ┌──────────────────────────┐
              │ joch Cloud Control   │
              │ Plane                    │
              └───────────┬──────────────┘
                          │ secure tunnel / API
                          ▼
              ┌──────────────────────────┐
              │ Customer Runtime Plane   │
              │ Kubernetes / Docker / VM │
              └──────────────────────────┘
```

This keeps private data and tool execution inside the customer environment.

---

# 6. Control plane vs data plane

This split is essential.

## Control plane

Owns:

```text
Specs
Policies
Resource registry
Auth/RBAC
Planning metadata
Deployments
Approvals
Budgets
Audit logs
Configuration
```

Services:

```text
joch-apiserver
joch-controller-manager
joch-policy-engine
joch-approval-service
joch-web-ui
```

## Data plane

Owns:

```text
Execution
Model calls
Tool calls
MCP servers
Memory reads/writes
RAG retrieval
Artifacts
Runtime traces
```

Services:

```text
joch-worker
joch-tool-gateway
joch-model-router
joch-memory
joch-rag
joch-artifact-service
joch-observability-agent
```

This lets users choose:

```text
Managed control plane, self-hosted data plane
Fully self-hosted
Fully local
Fully cloud
```

---

# 7. The universal runtime interface

To make Docker/Kubernetes/etc. pluggable, define a runtime abstraction:

```ts
interface RuntimeAdapter {
  createWorker(spec: WorkerSpec): Promise<WorkerHandle>;
  createJob(spec: JobSpec): Promise<JobHandle>;
  scaleDeployment(name: string, replicas: number): Promise<void>;
  getLogs(ref: RuntimeRef): AsyncIterable<LogEvent>;
  getStatus(ref: RuntimeRef): Promise<RuntimeStatus>;
  delete(ref: RuntimeRef): Promise<void>;
}
```

Implementations:

```text
LocalRuntimeAdapter
DockerRuntimeAdapter
DockerComposeRuntimeAdapter
KubernetesRuntimeAdapter
NomadRuntimeAdapter
ServerlessRuntimeAdapter
```

Then `DeploymentController` does not care where it runs.

It says:

```text
I need 3 workers for research-agent.
```

The adapter decides how:

```text
Local: spawn 3 processes
Docker: start 3 containers
Kubernetes: create Deployment with replicas: 3
Nomad: create job group count: 3
```

---

# 8. The universal resource store interface

Same idea for state.

```ts
interface ResourceStore {
  create(resource: Resource): Promise<Resource>;
  get(ref: ResourceRef): Promise<Resource>;
  list(selector: Selector): Promise<Resource[]>;
  updateSpec(ref: ResourceRef, spec: object): Promise<Resource>;
  updateStatus(ref: ResourceRef, status: object): Promise<Resource>;
  watch(selector: Selector): AsyncIterable<ResourceEvent>;
}
```

Backends:

```text
PostgresResourceStore
SQLiteResourceStore
KubernetesCRDResourceStore
CloudResourceStore
```

This is very powerful.

It means `joch` specs can live in:

```text
Local SQLite
Postgres
Kubernetes CRDs
Managed cloud
```

Same CLI. Same resource model.

---

# 9. Kubernetes integration options

we have two good ways to consume from Kubernetes.

## Option 1: Kubernetes as runtime only

`joch` stores resources in its own Postgres API server.

The Kubernetes adapter creates Pods/Jobs/Deployments.

```text
joch API Server
  ↓
Postgres Resource Store
  ↓
Kubernetes Runtime Adapter
  ↓
Kubernetes Jobs/Deployments
```

Pros:

```text
Works the same across Docker/local/Kubernetes
Simpler cross-platform design
Easier SaaS control plane
Full control over API semantics
```

Cons:

```text
Kubernetes users cannot manage everything with kubectl directly
Need our own API server
```

---

## Option 2: Kubernetes-native CRDs

`Agent`, `Execution`, `Tool`, etc. become Kubernetes CRDs.

```text
kubectl apply -f agent.yaml
  ↓
Kubernetes API Server
  ↓
joch Operator
  ↓
joch Runtime Services
```

Pros:

```text
Native Kubernetes UX
kubectl get agents
GitOps works naturally
RBAC/namespace model comes for free
Kubernetes watch/reconcile machinery comes for free
```

Cons:

```text
Ties production model to Kubernetes
Harder local/Docker story
Kubernetes CRDs are not always ideal for high-volume runtime events
```

---

## Recommended: support both

Use a common internal resource model, then support multiple stores:

```text
Local mode:
  SQLiteResourceStore

Docker mode:
  PostgresResourceStore

Kubernetes mode:
  KubernetesCRDResourceStore or Postgres + K8s adapter

Cloud mode:
  CloudResourceStore
```

This keeps the product portable.

---

# 10. How Kubernetes consumption should look

A Kubernetes user should be able to do both:

```bash
joch apply -f agent.yaml
```

and:

```bash
kubectl apply -f agent.yaml
```

Example CRD:

```yaml
apiVersion: joch.dev/v1alpha1
kind: Agent
metadata:
  name: research-agent
  namespace: ai
spec:
  modelRef:
    name: gpt-5-thinking
  personalityRef:
    name: pragmatic-researcher
  skills:
    - name: web-research
  tools:
    - name: web.search
    - name: github.search_issues
  memories:
    working:
      name: research-working-memory
  execution:
    loopRef:
      name: react-loop
    maxSteps: 30
```

Then:

```bash
kubectl get agents -n ai
kubectl describe agent research-agent -n ai
kubectl get executions -n ai
kubectl logs job/exec-123
```

But our CLI gives the better agent-native UX:

```bash
joch run research-agent "Research MCP server security"
joch trace exec-123
joch approvals ls
joch memories inspect research-working-memory
```

---

# 11. Docker consumption should look like this

Generate a Compose stack:

```bash
joch init docker-compose
joch up docker
```

Generated:

```yaml
services:
  joch-server:
    image: ghcr.io/acme/joch-server:latest
    environment:
      joch_STORE: postgres
      joch_DATABASE_URL: postgres://joch:joch@postgres:5432/joch
      joch_EVENT_BUS: redis
    ports:
      - "8080:8080"

  joch-worker:
    image: ghcr.io/acme/joch-worker:latest
    environment:
      joch_SERVER_URL: http://joch-server:8080
      joch_WORKER_POOL: default
    depends_on:
      - joch-server

  joch-tool-gateway:
    image: ghcr.io/acme/joch-tool-gateway:latest
    environment:
      joch_SERVER_URL: http://joch-server:8080

  joch-memory:
    image: ghcr.io/acme/joch-memory:latest
    environment:
      DATABASE_URL: postgres://joch:joch@postgres:5432/joch
      VECTOR_STORE: pgvector

  postgres:
    image: pgvector/pgvector:pg16

  redis:
    image: redis:7

  minio:
    image: minio/minio
```

Then users can run:

```bash
joch context set docker --server http://localhost:8080
joch apply -f examples/research-agent.yaml
joch run research-agent "Summarize this repo"
```

---

# 12. Kubernetes deployment should look like this

Install:

```bash
helm install joch joch/joch \
  --namespace joch-system \
  --create-namespace
```

This installs:

```text
joch-apiserver Deployment
joch-controller-manager Deployment
joch-worker Deployment
joch-tool-gateway Deployment
joch-memory Deployment
joch-rag Deployment
joch-model-router Deployment
CRDs
RBAC
Services
NetworkPolicies
ConfigMaps
Secrets
```

Then:

```bash
kubectl apply -f agent.yaml
```

Or:

```bash
joch apply -f agent.yaml --context prod
```

---

# 13. Service APIs

Each service should expose both internal and external APIs where appropriate.

## API Server

```text
POST /v1/resources/apply
GET  /v1/namespaces/{ns}/agents
GET  /v1/namespaces/{ns}/executions/{name}
PATCH /v1/namespaces/{ns}/executions/{name}/status
WATCH /v1/resources
```

## Execution API

```text
POST /v1/executions
GET  /v1/executions/{id}
POST /v1/executions/{id}/cancel
GET  /v1/executions/{id}/events
```

## Tool Gateway API

```text
POST /v1/tool-calls
GET  /v1/tool-calls/{id}
POST /v1/mcp/discover
```

## Memory API

```text
POST /v1/memory/query
POST /v1/memory/write
DELETE /v1/memory/items/{id}
POST /v1/memory/summarize
```

## RAG API

```text
POST /v1/rag/retrieve
POST /v1/rag/index
GET  /v1/rag/sources/{id}/status
```

## Model Router API

```text
POST /v1/model/respond
POST /v1/model/stream
GET  /v1/models/capabilities
```

## Policy API

```text
POST /v1/policy/authorize
POST /v1/policy/evaluate
```

---

# 14. Internal event model

Services should communicate through events.

```text
ResourceApplied
ResourceValidated
AgentCompiled
ExecutionCreated
ExecutionScheduled
ExecutionStarted
ModelCallStarted
ModelCallCompleted
ToolCallRequested
ToolCallApproved
ToolCallCompleted
MemoryWritten
RAGRetrieved
ArtifactCreated
ExecutionSucceeded
ExecutionFailed
BudgetExceeded
PolicyDenied
```

This makes services loosely coupled.

Example event:

```json
{
  "type": "ToolCallRequested",
  "execution": "exec-123",
  "agent": "research-agent",
  "tool": "github.create_issue",
  "sideEffect": "external_write",
  "approvalRequired": true,
  "timestamp": "2026-05-10T10:00:00Z"
}
```

---

# 15. The adapter pattern

This should be a central internal concept.

```text
Provider adapters:
  OpenAIAdapter
  AnthropicAdapter
  GoogleAdapter
  LocalModelAdapter

Runtime adapters:
  LocalAdapter
  DockerAdapter
  KubernetesAdapter
  NomadAdapter

Store adapters:
  SQLiteStore
  PostgresStore
  KubernetesCRDStore

Secret adapters:
  EnvSecretProvider
  KubernetesSecretProvider
  VaultSecretProvider
  AWSSecretsManagerProvider

Artifact adapters:
  LocalFSArtifactStore
  S3ArtifactStore
  GCSArtifactStore
  MinIOArtifactStore

Queue adapters:
  PostgresQueue
  RedisQueue
  NATSQueue
  SQSQueue

Vector adapters:
  PgVectorAdapter
  PineconeAdapter
  WeaviateAdapter
  QdrantAdapter
```

This is how we avoid hard-coding infrastructure assumptions.

---

# 16. Recommended repository layout

```text
joch/
  cmd/
    joch/
    joch-server/
    joch-worker/
    joch-gateway/
    joch-memory/
    joch-operator/

  api/
    openapi/
    proto/
    crds/

  pkg/
    apis/
    store/
    admission/
    controllers/
    compiler/
    runtime/
    scheduler/
    models/
    tools/
    mcp/
    memory/
    rag/
    policy/
    approvals/
    artifacts/
    observability/
    adapters/
      runtime/
        local/
        docker/
        kubernetes/
      store/
        sqlite/
        postgres/
        k8scrd/
      secrets/
        env/
        vault/
        kubernetes/
      models/
        openai/
        anthropic/
        google/
      vector/
        pgvector/
        qdrant/

  deploy/
    docker-compose/
    helm/
    kubernetes/
    local/

  examples/
    agents/
    models/
    mcp/
    memory/
    rag/
```

---

# 17. How an agent execution works in this architecture

```text
1. User submits Agent, Model, Tool, Memory specs.
2. API Server validates and stores specs.
3. Controller Manager reconciles resources.
4. Compiler creates CompiledAgentManifest.
5. User creates Execution.
6. Scheduler assigns execution to a worker.
7. Worker loads compiled manifest.
8. Worker calls Model Router.
9. Model returns a tool-call request.
10. Worker creates ToolCall.
11. Tool Gateway authorizes and executes it.
12. Memory/RAG services provide context.
13. Artifact Service stores output.
14. Observability records trace.
15. Execution status becomes Succeeded.
```

This flow is identical whether the worker is:

```text
a local process
a Docker container
a Kubernetes Job
a VM service
a serverless function
```

Only the deployment adapter changes.

---

# 18. The critical internal artifact: compiled manifest

The specs are human-authored. The runtime should consume a compiled manifest.

```yaml
apiVersion: internal.joch.dev/v1
kind: CompiledAgentManifest
metadata:
  name: research-agent-g12
spec:
  agent:
    name: research-agent
    generation: 12

  model:
    provider: openai
    model: gpt-5-thinking
    capabilities:
      toolCalling: true
      structuredOutput: true

  prompt:
    system: |
      we are a pragmatic research agent...
    personalityChecksum: sha256:abc123

  tools:
    - name: web.search
      gatewayUrl: http://joch-tool-gateway
      sideEffect: read_only
      schema:
        type: object

  memory:
    working:
      endpoint: http://joch-memory
      namespace: default
      name: research-working-memory

  rag:
    - endpoint: http://joch-rag
      name: company-docs-rag

  policies:
    resolved:
      - no-external-write-without-approval
      - citation-required

  guardrails:
    output:
      - citation-required

  observability:
    traceEndpoint: http://joch-observability
```

Workers should run from this, not from raw YAML.

---

# 19. Docker-friendly packaging

Each major service should be a container image:

```text
ghcr.io/acme/joch-server
ghcr.io/acme/joch-worker
ghcr.io/acme/joch-gateway
ghcr.io/acme/joch-memory
ghcr.io/acme/joch-rag
ghcr.io/acme/joch-observability
ghcr.io/acme/joch-operator
```

All services should support config through:

```text
environment variables
config files
CLI flags
Kubernetes ConfigMaps
Helm values
```

Example:

```bash
joch-server \
  --store=postgres \
  --database-url=$DATABASE_URL \
  --event-bus=redis \
  --auth=oidc
```

---

# 20. Kubernetes-friendly packaging

Provide:

```text
Helm chart
CRDs
Operator
RBAC templates
NetworkPolicies
ServiceMonitor
OpenTelemetry config
Example values files
```

Example Helm values:

```yaml
joch:
  mode: selfHosted

server:
  replicas: 2
  store:
    type: postgres

worker:
  replicas: 3
  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 20

toolGateway:
  enabled: true

memory:
  enabled: true
  backend: postgres

rag:
  enabled: true
  vectorStore: pgvector

observability:
  otel:
    enabled: true

runtime:
  adapter: kubernetes
```

---

# 21. Local-friendly packaging

For developers:

```bash
brew install joch
joch up
```

This could start:

```text
joch-server in-process
SQLite
local worker
local gateway
local file artifact store
```

Useful command:

```bash
joch doctor
```

Checks:

```text
API server reachable
Model credentials configured
MCP servers reachable
Memory backend ready
Worker registered
```

---

# 22. Multi-environment contexts

The CLI should support contexts like `kubectl`.

```bash
joch context ls
joch context use local
joch context use docker
joch context use prod-k8s
joch context use cloud
```

Each context defines:

```yaml
name: prod-k8s
server: https://joch.example.com
auth:
  type: oidc
namespace: ai-prod
runtime:
  type: kubernetes
```

---

# 23. Consumption from CI/CD

The platform should fit GitOps and CI.

```bash
joch validate -f .
joch diff -f .
joch apply -f .
joch eval run research-agent-eval
joch deploy research-agent --env staging
joch promote research-agent --from staging --to prod
```

GitHub Actions:

```yaml
- name: Validate agent specs
  run: joch validate -f ./joch

- name: Run evals
  run: joch eval run -f ./joch/evals

- name: Deploy
  run: joch apply -f ./joch
```

Kubernetes GitOps:

```text
Argo CD / Flux applies Agent CRDs.
joch operator reconciles them.
```

---

# 24. Security boundary

For portable services, be very clear about trust zones.

```text
Control plane:
  stores specs and policy

Runtime plane:
  executes model/tool/memory operations

Tool gateway:
  only place where external side effects happen

Secret broker:
  only place where credentials are resolved

Model router:
  only place where provider calls happen

Memory/RAG:
  only place where durable context is read/written
```

Do **not** let arbitrary workers directly access:

```text
provider API keys
MCP credentials
database passwords
raw secrets
```

Workers should request operations from services.

---

# 25. Network model

In Docker:

```text
joch-worker → joch-server
joch-worker → joch-model-router
joch-worker → joch-tool-gateway
joch-worker → joch-memory
```

In Kubernetes:

```text
worker ServiceAccount can call:
  model-router
  tool-gateway
  memory
  rag
  observability

worker cannot directly call:
  Vault
  external DB
  sensitive internal services
```

Use NetworkPolicies in Kubernetes to enforce this.

---

# 26. What should be pluggable?

Make these interfaces pluggable from day one:

```text
Resource store
Runtime adapter
Model provider
Tool provider
MCP transport
Memory backend
Vector store
Artifact store
Secret provider
Policy backend
Queue/event bus
Observability exporter
Auth provider
```

Do **not** make the core resource model pluggable too early. That should stay stable.

---

# 27. MVP shape

For MVP, I would build:

```text
joch CLI

joch-server
  API server
  resource store
  admission
  policy-lite
  controller manager
  compiler

joch-worker
  execution engine
  simple planner
  model adapters
  prompt compiler

joch-gateway
  MCP/tool calls
  side-effect audit

joch-memory
  working memory + pgvector RAG

Postgres
  specs, status, events, memory metadata

Object storage
  artifacts

OpenTelemetry-compatible logs/traces
```

Supported deployment modes:

```text
local
docker compose
kubernetes via Helm
```

Supported specs:

```text
Agent
Model
Personality
Prompt
Tool
MCPServer
Memory
RAG
Execution
Policy
Trace
Artifact
```

That is enough to prove the platform.

---

# 28. Suggested binary layout

```text
joch
  CLI

joch-server
  control plane

joch-worker
  execution runtime

joch-gateway
  tools and MCP

joch-memory
  memory and RAG

joch-operator
  Kubernetes operator
```

Later split:

```text
joch-model-router
joch-policy-engine
joch-approval-service
joch-eval-service
joch-observability
```

---

# 29. What the user experience should look like

## Local

```bash
joch up
joch apply -f examples/research-agent.yaml
joch run research-agent "Find competitors to joch"
joch trace last
```

## Docker

```bash
joch init docker-compose
docker compose up -d
joch context use docker
joch apply -f ./joch
```

## Kubernetes

```bash
helm install joch joch/joch -n joch-system --create-namespace
kubectl apply -f ./joch
joch context use prod
joch get agents
joch get executions
```

## SaaS + private runtime

```bash
joch cloud login
joch runtime install kubernetes --name prod-eu
joch apply -f ./joch --context cloud
```

---

# 30. The cleanest final architecture
