# Repository Strategy

PeasantsAI ships Joch through a focused multi-repo layout. The project starts with a small set of repositories that grow as boundaries harden, rather than splitting prematurely.

## Repositories now

| Repository | Purpose |
|---|---|
| `peasantsai/joch` | Main control-plane monorepo: CLI, server, worker, gateways, router, memory, trace, operator, framework adapters, provider adapters. |
| `peasantsai/joch-spec` | OpenAPI / Proto / JSON Schema definitions for resources; CRDs; example YAML; RFC archive. |
| `peasantsai/joch-docs` | This documentation site. |
| `peasantsai/joch-examples` | Runnable example agents across SDKs and runtimes. |
| `peasantsai/joch-registry` | Open registry of reusable policies, tools, MCP server definitions, and agent templates. |

## Repositories later

| Repository | Triggers the split |
|---|---|
| `peasantsai/joch-operator` | When Kubernetes integration warrants its own release cadence and CRD versioning. |
| `peasantsai/joch-mcp` | When the official MCP servers and gateway helpers grow beyond the core. |
| `peasantsai/joch-js` | When the TypeScript client SDK and console SDK warrant their own release cadence. |
| `peasantsai/joch-python` | Python SDK for agent runtimes and eval pipelines. |
| `peasantsai/joch-ui` | Joch Console (web app). Initially lives inside `peasantsai/joch`. |
| `peasantsai/joch-adapters` | When provider, runtime, vector, secret, or artifact adapter volume justifies its own home. |
| `peasantsai/joch-cloud` | Private repo for the multi-tenant hosted control plane. |

## Main monorepo layout

```text
peasantsai/joch/
  cmd/
    joch/             # CLI
    joch-server/      # control plane
    joch-worker/      # runtime worker
    joch-gateway/     # tool + MCP gateway
    joch-router/      # model router
    joch-memory/      # memory + RAG
    joch-trace/       # trace ingest + export
    joch-operator/    # Kubernetes operator (until split)
  api/
    openapi/
    proto/
    crds/
    schemas/
  pkg/
    apiserver/
    admission/
    auth/
    store/
    controllers/
    compiler/
    scheduler/
    runtime/
    models/
    tools/
    mcp/
    memory/
    rag/
    policy/
    approvals/
    artifacts/
    agbom/
    observability/
    adapters/
      framework/      # one per SDK
      runtime/        # local, docker, kubernetes
      provider/       # openai, anthropic, google, ms, ollama, vllm
      store/          # sqlite, postgres, k8scrd
      secrets/        # env, vault, kubernetes, aws-sm
      vector/         # pgvector, qdrant, pinecone, weaviate
  deploy/
    compose/
    helm/
    kubernetes/
    local/
  examples/
    basic/
    rag/
    mcp/
    multi-agent/
  docs/
    dev/
    architecture/
  scripts/
  tests/
```

## Spec repository layout

```text
peasantsai/joch-spec/
  schemas/
    agent.schema.json
    framework-adapter.schema.json
    model.schema.json
    model-route.schema.json
    tool.schema.json
    mcpserver.schema.json
    policy.schema.json
    approval.schema.json
    agbom.schema.json
    trace.schema.json
    execution.schema.json
    conversation.schema.json
    state-checkpoint.schema.json
    memory.schema.json
    rag.schema.json
    knowledge-source.schema.json
    artifact.schema.json
    secret.schema.json
    budget.schema.json
    eval.schema.json
    deployment.schema.json
    environment.schema.json
    team.schema.json
    handoff.schema.json
  crds/
    joch.dev_agents.yaml
    joch.dev_models.yaml
    joch.dev_executions.yaml
    ...
  examples/
    basic-agent.yaml
    rag-agent.yaml
    mcp-agent.yaml
    multi-agent-system.yaml
  rfcs/
    0001-resource-model.md
    0002-execution-model.md
    0003-portable-policy.md
    0004-agbom.md
    0005-aos-conformance.md
```

## Registry repository layout

```text
peasantsai/joch-registry/
  agents/                 # reusable agent templates
  policies/               # reusable Policy resources
  tools/                  # reusable Tool definitions
  mcpservers/             # vetted MCPServer entries
  rag-templates/
  eval-packs/
```

The registry is the seed of the future Joch Marketplace.

## Naming inside the product

```text
Joch Server
Joch Worker
Joch Gateway        (tool + MCP)
Joch Router         (model)
Joch Memory
Joch Trace
Joch Console
Joch Operator
Joch Registry
Joch Marketplace
Joch Cloud
Joch Enterprise
```

Use these names consistently across docs, the CLI, container image tags, and marketing.

## CLI examples

```bash
joch up
joch apply -f .
joch get agents
joch describe agent support-triage
joch run support-triage "Triage incoming queue"
joch trace exec-20260510-001
joch approvals ls
joch mcp ls
joch agbom support-triage
joch registry search policies
joch promote support-triage --from staging --to prod
```
