# Control Plane

The Joch control plane is a domain-specific control plane for AI agent fleets. It owns desired state — agents, models, tools, MCP servers, policies, deployments, environments — and reconciles that state into runtime configuration. It does **not** own the agent loop; that lives in whichever SDK built the agent.

## Services

The control plane is a small set of well-bounded services:

```text
joch-apiserver           REST + gRPC + Watch API for resources
joch-resource-store      durable storage (SQLite | Postgres | Kubernetes CRDs)
joch-admission           validates and defaults resource specs on apply
joch-policy-engine       evaluates policies against decision contexts
joch-controller-manager  reconciles desired state into runtime artifacts
joch-compiler            compiles agent records into CompiledAgentManifests
joch-scheduler           assigns executions to workers
joch-approval-service    routes approval requests; records decisions
joch-budget-service      tracks cost and enforces caps
joch-eval-service        runs evals on a schedule, on demand, on promotion
joch-abom-service        generates and refreshes per-agent ABOM
joch-secret-broker       resolves Secret references at the right boundary
```

In small deployments these collapse into a single `joch-server` binary. In large deployments they split out for independent scaling.

## Resource store

The store is pluggable. Joch ships three backends:

```text
SQLite           local mode, single-process developer experience
Postgres         shared resource store for Docker / VM / SaaS deployments
Kubernetes CRDs  stores resources as native CRDs for kubectl-friendly use
```

The same resource catalog ([Resources](../specs/index.md)) works against all three backends. Operators choose by context, not by changing the spec.

## Apply, validate, reconcile

An apply pipeline is identical across backends:

```text
joch apply -f agent.yaml
        │
        ▼
joch-apiserver  (auth, RBAC, audit)
        │
        ▼
joch-admission  (schema, defaults, cross-references, policy preflight)
        │
        ▼
joch-resource-store  (versioned put)
        │
        ▼
joch-controller-manager  (reconcile)
        │
        ▼
joch-compiler  (CompiledAgentManifest)
        │
        ▼
runtime adapter  (Local | Docker | Kubernetes | …)
```

If the apply fails policy preflight (e.g., a referenced model is denied for the target environment), the apply rejects with a clear diagnostic. Nothing is partially-applied.

## Compiled agent manifests

Specs are human-authored. The runtime consumes a deterministic, fully-resolved manifest:

```yaml
apiVersion: internal.joch.dev/v1
kind: CompiledAgentManifest
metadata:
  name: support-triage-g14
spec:
  agent:
    name: support-triage
    generation: 14
  framework:
    adapter: openai-agents-sdk
    adapterImage: ghcr.io/peasantsai/joch-adapter-openai:1.0.0
  modelRoute:
    primary: openai:gpt-5-thinking
    fallback:
      - anthropic:claude-sonnet
  tools:
    - name: zendesk.search
      gatewayUrl: http://joch-tool-gateway
      sideEffect: read_only
      schemaSha: sha256:abc...
    - name: slack.send
      gatewayUrl: http://joch-tool-gateway
      sideEffect: external_write
      requiresApproval: true
  mcpServers:
    - name: github
      gatewayUrl: http://joch-mcp-gateway
      pinnedVersion: 1.2.0
      trustScore: 0.9
  memories:
    working:
      endpoint: http://joch-memory
      namespace: support-platform
      name: support-triage-working
  policies:
    resolved:
      - no-customer-data-exfiltration@v3
      - external-send-requires-approval@v2
  observability:
    traceEndpoint: http://joch-trace
    abomEndpoint: http://joch-abom
```

Workers run from the manifest, not from raw YAML. The compiler is the single place where references are resolved and capability checks happen.

## Authentication and authorization

The control plane authenticates clients via OIDC, with optional integrations for Kubernetes service accounts. Authorization uses roles bound to teams / namespaces. Every API call lands in the audit log with the operator identity, the resource touched, and the effective policy.

## Multi-tenancy

Joch's multi-tenant model is `Team / Namespace` plus `Environment`:

- **Team / Namespace** — ownership boundary; roles are bound here.
- **Environment** — promotion boundary (`dev`, `staging`, `prod`).
- **Policy selectors** — bind policies to namespaces or environments.

For SaaS deployments, an additional `Tenant` boundary wraps namespaces. See [Trust and Security Model](trust-and-security-model.md).

## API surface

```text
POST   /v1/resources/apply
GET    /v1/namespaces/{ns}/agents
GET    /v1/namespaces/{ns}/agents/{name}
GET    /v1/namespaces/{ns}/agents/{name}/abom
GET    /v1/namespaces/{ns}/executions
POST   /v1/namespaces/{ns}/executions
GET    /v1/namespaces/{ns}/toolcalls
GET    /v1/namespaces/{ns}/approvals
POST   /v1/namespaces/{ns}/approvals/{id}/decide
POST   /v1/eval/{name}/run
POST   /v1/promote
WATCH  /v1/resources
```

The full API is published as OpenAPI in the [`peasantsai/joch-spec`](https://github.com/peasantsai/joch-spec) repository.

## What the control plane does *not* do

- It does not run model calls. Those go through the [Model Router](model-router.md).
- It does not run tool calls. Those go through the [Tool Gateway](tool-gateway.md).
- It does not store conversation transcripts. Those live in the [`Conversation`](../specs/kubernetes/conversation.md) resource backed by the data plane.
- It does not run agent loops. Those live in the SDK runtime via a [framework adapter](framework-adapters.md).

This separation keeps the control plane small, deterministic, and easy to audit.
