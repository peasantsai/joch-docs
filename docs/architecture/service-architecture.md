# Service Architecture

Joch is a control plane plus a thin data plane, packaged as a small set of Go and Python services. The shape is identical across local, Docker, and Kubernetes runtimes — only the deployment adapter changes.

## Layered view

```text
┌────────────────────────────────────────────────────────────────┐
│  Layer 1 — Operator surface                                    │
│  joch CLI, web console, REST/gRPC, OpenAPI/Proto SDKs          │
└──────────────────────────┬─────────────────────────────────────┘
                           ▼
┌────────────────────────────────────────────────────────────────┐
│  Layer 2 — Control plane services                              │
│  apiserver, resource store, admission, policy engine,          │
│  controller manager, compiler, scheduler, approval service,    │
│  budget service, eval service, ABOM service                    │
└──────────────────────────┬─────────────────────────────────────┘
                           ▼
┌────────────────────────────────────────────────────────────────┐
│  Layer 3 — Data plane services                                 │
│  tool gateway, mcp gateway, model router,                      │
│  memory, rag, artifact, trace, a2a-broker, runtime worker      │
└──────────────────────────┬─────────────────────────────────────┘
                           ▼
┌────────────────────────────────────────────────────────────────┐
│  Layer 4 — Deployment adapters                                 │
│  local, docker, docker-compose, kubernetes, nomad, serverless  │
└────────────────────────────────────────────────────────────────┘
```

Layer 2 and 3 services are the same binaries across Layer 4. Adapters change only how they are scheduled, networked, and secret-bound.

## Initial binary layout

For v1, services collapse into a small set of binaries to avoid premature micro-servicing:

```text
joch                CLI
joch-server         apiserver + admission + store + policy engine + controller mgr + compiler
joch-worker         runtime worker (loads framework adapters)
joch-gateway        tool gateway + MCP gateway + secret broker
joch-memory         memory + RAG + artifact
joch-router         model router
joch-trace          trace ingest + export
joch-operator       Kubernetes operator (CRDs + controllers)
```

Later, as scale demands, the natural splits are: model router, policy engine, approval service, eval service, ABOM service, observability.

## Pluggable interfaces

Five interfaces are pluggable from day one:

```text
Resource store        SQLite | Postgres | Kubernetes CRDs | Cloud
Runtime adapter       Local | Docker | Kubernetes | Nomad | Serverless
Secret provider       Env | Vault | Kubernetes secrets | AWS Secrets Manager
Vector store          pgvector | Qdrant | Pinecone | Weaviate
Artifact store        Local FS | S3 | GCS | MinIO | Azure Blob
```

The core resource model is **not** pluggable; it is the stable contract.

## Deployment modes

### Local

```bash
brew install joch
joch up
joch apply -f examples/support-triage.yaml
joch run support-triage "Triage incoming Zendesk queue"
```

Under the hood: `joch-server` in-process, SQLite store, local worker subprocess, local artifact filesystem, no Kubernetes.

### Docker Compose

```bash
joch init docker-compose
docker compose up -d
joch context use docker
joch apply -f .
```

Compose stack: `joch-server`, `joch-worker`, `joch-gateway`, `joch-memory`, `joch-router`, `joch-trace`, Postgres (with pgvector), Redis (event bus), MinIO (artifacts), OpenTelemetry collector.

### Kubernetes

```bash
helm install joch joch/joch -n joch-system --create-namespace
kubectl apply -f .
```

Installed: CRDs, Helm chart, Operator, Deployments per service, Services, NetworkPolicies, ServiceAccounts, RBAC, OpenTelemetry config, optional ServiceMonitor for Prometheus.

### Managed control plane + customer runtime

```text
                 ┌──────────────────────────┐
                 │ Joch Cloud Control Plane │
                 │ specs, UI, policies,     │
                 │ traces, evals, ABOM      │
                 └─────────────┬────────────┘
                               │ secure tunnel / API
                               ▼
                 ┌──────────────────────────┐
                 │ Customer runtime plane   │
                 │ workers, gateways,       │
                 │ memory, RAG, secrets     │
                 └──────────────────────────┘
```

Private data and tool execution stay inside the customer environment. Joch Cloud holds the cross-tenant inventory and observability surfaces.

## Universal runtime interface

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

Implementations: `LocalRuntimeAdapter`, `DockerRuntimeAdapter`, `KubernetesRuntimeAdapter`, `NomadRuntimeAdapter`, `ServerlessRuntimeAdapter`. The deployment controller speaks only the adapter interface.

## Universal resource store interface

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

Backends: `SQLiteResourceStore`, `PostgresResourceStore`, `KubernetesCRDResourceStore`, `CloudResourceStore`. The CLI is identical against any backend.

## Service APIs

```text
apiserver           POST /v1/resources/apply, WATCH /v1/resources, ...
tool gateway        POST /v1/tool-calls, GET /v1/tool-calls/{id}
mcp gateway         POST /v1/mcp/discover, POST /v1/mcp/call, ...
model router        POST /v1/model/respond, POST /v1/model/stream
memory              POST /v1/memory/query, POST /v1/memory/write
rag                 POST /v1/rag/retrieve, POST /v1/rag/index
trace               OTLP / OCSF ingest, query API
approval            POST /v1/approvals, POST /v1/approvals/{id}/decide
abom                GET /v1/abom/{agent}, POST /v1/abom/{agent}/refresh
```

Full OpenAPI / Proto definitions live in [`peasantsai/joch-spec`](https://github.com/peasantsai/joch-spec).

## Internal event model

Services exchange events instead of synchronous calls where possible:

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
MemoryRead
MemoryWritten
KnowledgeRetrieved
ArtifactCreated
HookDecision
PolicyDenied
ApprovalRequested
ApprovalGranted
ApprovalDenied
A2AMessageSent
A2AMessageReceived
ProviderSwitched
ABOMUpdated
ExecutionSucceeded
ExecutionFailed
BudgetExceeded
```

The event bus backend is pluggable: Postgres `LISTEN/NOTIFY`, Redis Streams, NATS JetStream, AWS SQS, Kafka.

## Repository layout

The main monorepo (`peasantsai/joch`) is laid out as:

```text
cmd/
  joch/          joch-server/      joch-worker/
  joch-gateway/  joch-memory/      joch-router/
  joch-trace/    joch-operator/

api/
  openapi/       proto/      crds/

pkg/
  apiserver/     admission/    auth/         store/
  controllers/   compiler/     scheduler/    runtime/
  models/        tools/        mcp/          memory/
  rag/           policy/       approvals/    artifacts/
  observability/ adapters/

deploy/
  docker-compose/   helm/   kubernetes/   local/

examples/        scripts/     tests/      docs/
```

See [Repository Strategy](../product/repositories.md) for the multi-repo split (spec, examples, registry, MCP, SDKs, UI).

## Why this architecture is durable

- **Agent semantics are owned by Joch.** Infrastructure execution is delegated to adapters.
- **The data plane is small** — five core services and one runtime worker — so the audit surface is bounded.
- **The control plane is replaceable** in a single namespace by reapplying records to a different store.
- **Adding an SDK is a package**, not a fork.
- **Adding a provider is a package**, not a fork.

The architecture survives a change of SDK, provider, store backend, or runtime by design.
