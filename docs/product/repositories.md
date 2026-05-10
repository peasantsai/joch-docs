# Repository Strategy

# GitHub repo structure for `github.com/peasantsai`

I would not create 30 repos immediately. Start with a focused multi-repo architecture that can grow.

## Core repos to create first

```text
github.com/peasantsai/joch
```

Main monorepo. Owns the core product.

Contains:

```text
cmd/
  joch/
  joch-server/
  joch-worker/
  joch-gateway/
  joch-operator/

pkg/
  apis/
  controllers/
  compiler/
  runtime/
  models/
  tools/
  memory/
  rag/
  policy/
  observability/
  adapters/

deploy/
  docker-compose/
  helm/
  kubernetes/

examples/
docs/
```

This should be the main repo until the architecture stabilizes.

---

```text
github.com/peasantsai/joch-spec
```

Owns the resource specifications.

Contains:

```text
schemas/
  agent.schema.json
  model.schema.json
  skill.schema.json
  personality.schema.json
  tool.schema.json
  mcpserver.schema.json
  memory.schema.json
  rag.schema.json
  execution.schema.json
  policy.schema.json

crds/
  joch.dev_agents.yaml
  joch.dev_models.yaml
  joch.dev_executions.yaml

examples/
  basic-agent.yaml
  rag-agent.yaml
  multi-agent-system.yaml

rfcs/
  0001-resource-model.md
  0002-execution-model.md
  0003-memory-model.md
```

Why separate it:

```text
Specs need to be stable, versioned, and consumable by other projects.
```

---

```text
github.com/peasantsai/joch-docs
```

Documentation site.

Contains:

```text
docs/
  getting-started/
  concepts/
  reference/
  tutorials/
  deployment/
  api/

website/
```

Could use Mintlify, Docusaurus, Astro, or VitePress.

---

```text
github.com/peasantsai/joch-examples
```

Example agents and templates.

Contains:

```text
examples/
  personal-research-agent/
  github-triage-agent/
  customer-support-agent/
  data-analyst-agent/
  code-review-agent/
  multi-agent-research-team/
  rag-over-company-docs/
  mcp-github-agent/
  k8s-deployed-agent/
```

This is useful for adoption and demos.

---

```text
github.com/peasantsai/joch-operator
```

Kubernetes operator, eventually split from the monorepo.

Owns:

```text
CRD controllers
Helm chart
Kubernetes runtime adapter
Admission webhooks
Cluster installation
```

Start this inside `joch`, then split it later when Kubernetes support becomes substantial.

---

```text
github.com/peasantsai/joch-js
```

JavaScript/TypeScript SDK.

For:

```text
Node.js users
Web dashboards
CLI plugins
Agent app developers
```

Package:

```text
@joch/sdk
```

Or under the organization:

```text
@peasantsai/joch
```

---

```text
github.com/peasantsai/joch-python
```

Python SDK.

For:

```text
AI engineers
Notebook users
Python agent runtimes
Eval pipelines
RAG integrations
```

Package:

```text
joch-ai
```

or:

```text
peasantsai-joch
```

---

```text
github.com/peasantsai/joch-registry
```

Public registry of reusable components.

Contains:

```text
agents/
personalities/
skills/
tools/
mcpservers/
execution-loops/
guardrails/
policies/
rag-templates/
```

Example:

```text
registry/
  personalities/
    pragmatic-researcher/
    senior-code-reviewer/
    customer-support-rep/

  skills/
    web-research/
    github-triage/
    report-writing/

  policies/
    no-external-write-without-approval/
    pii-redaction/
    citation-required/
```

This could later become our marketplace.

---

```text
github.com/peasantsai/joch-mcp
```

Official MCP servers and MCP integration helpers.

Contains:

```text
servers/
  filesystem/
  github/
  postgres/
  slack/
  browser/
  shell-sandbox/

gateway/
  mcp-proxy
  mcp-discovery
  mcp-schema-normalizer
```

This repo can become strategically important because MCP is one of our main interoperability layers.

---

```text
github.com/peasantsai/joch-adapters
```

Provider/runtime/storage adapters.

Contains:

```text
models/
  openai/
  anthropic/
  google/
  ollama/
  vllm/

runtime/
  local/
  docker/
  kubernetes/
  nomad/

memory/
  postgres/
  redis/
  sqlite/

vector/
  pgvector/
  qdrant/
  pinecone/
  weaviate/

secrets/
  env/
  vault/
  kubernetes/
  aws-secrets-manager/
```

These can also stay in the main repo at first, then split later.

---

```text
github.com/peasantsai/joch-ui
```

Web dashboard.

Owns:

```text
Agent inventory
Execution traces
Tool-call audit
Approvals
Memory browser
RAG indexing status
Cost dashboards
Deployment health
```

This can become:

```text
joch Console
```

---

```text
github.com/peasantsai/joch-cloud
```

Private repo if we plan to build SaaS.

Owns:

```text
multi-tenant control plane
billing
org management
hosted API
cloud UI
managed runtimes
customer runtime tunnels
```

I would keep this private.

---

# Minimal repo set to start

Do **not** start with too many repos.

Start with these:

```text
peasantsai/joch
peasantsai/joch-spec
peasantsai/joch-docs
peasantsai/joch-examples
peasantsai/joch-registry
```

Then add:

```text
peasantsai/joch-js
peasantsai/joch-python
peasantsai/joch-operator
peasantsai/joch-mcp
peasantsai/joch-ui
```

when the boundaries become real.

---

# Recommended initial repo list

Here is the clean version.

## 1. `peasantsai/joch`

Main platform monorepo.

```text
CLI
API server
controller manager
runtime worker
tool gateway
memory service
model router
core adapters
local mode
Docker mode
Kubernetes adapter starter
```

---

## 2. `peasantsai/joch-spec`

Resource model and schemas.

```text
Agent
Model
Skill
Personality
Prompt
Tool
MCPServer
Memory
RAG
KnowledgeSource
Plan
Execution
Deployment
Policy
Guardrail
Approval
Trace
Artifact
Eval
Budget
```

---

## 3. `peasantsai/joch-docs`

Documentation website.

```text
Getting started
Concepts
CLI reference
Spec reference
Architecture
Deployment guides
Tutorials
```

---

## 4. `peasantsai/joch-examples`

Runnable examples.

```text
Local examples
Docker Compose examples
Kubernetes examples
MCP examples
RAG examples
Multi-agent examples
```

---

## 5. `peasantsai/joch-registry`

Reusable agent components.

```text
Agent templates
Personality templates
Skill templates
Tool definitions
Policy packs
Guardrail packs
Execution loop templates
```

---

## 6. `peasantsai/joch-operator`

Kubernetes operator.

```text
CRDs
controllers
Helm chart
admission webhooks
Kubernetes runtime adapter
```

Could start inside `joch`, then split.

---

## 7. `peasantsai/joch-mcp`

MCP integration layer.

```text
MCP gateway
MCP discovery
official MCP servers
schema normalization
tool safety wrappers
```

---

## 8. `peasantsai/joch-js`

TypeScript SDK.

```text
Client SDK
resource builders
CLI extension helpers
web dashboard client
```

---

## 9. `peasantsai/joch-python`

Python SDK.

```text
Client SDK
agent runtime helpers
eval helpers
notebook support
```

---

## 10. `peasantsai/joch-ui`

Web console.

```text
Dashboard
traces
approvals
agent inventory
memory/RAG browser
tool-call audit
deployment status
```

---

## 11. `peasantsai/joch-adapters`

Optional later repo for external integrations.

```text
model providers
runtime providers
vector stores
secret stores
artifact stores
queue/event buses
```

Do this only once adapter volume is high.

---

## 12. `peasantsai/joch-cloud`

Private SaaS repo.

```text
hosted control plane
billing
multi-tenancy
cloud UI
orgs/users
managed runtime registration
```

---

# Repo creation order

I would create them in this order:

```text
1. peasantsai/joch
2. peasantsai/joch-spec
3. peasantsai/joch-docs
4. peasantsai/joch-examples
5. peasantsai/joch-registry
6. peasantsai/joch-operator
7. peasantsai/joch-mcp
8. peasantsai/joch-js
9. peasantsai/joch-python
10. peasantsai/joch-ui
11. peasantsai/joch-adapters
12. peasantsai/joch-cloud
```

But initially only actively develop:

```text
peasantsai/joch
peasantsai/joch-spec
peasantsai/joch-docs
peasantsai/joch-examples
```

---

# What goes in the main `joch` repo

Recommended layout:

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
    observability/
    adapters/

  deploy/
    docker-compose/
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

---

# Naming inside the product

Use these names consistently:

```text
joch Server
joch Worker
joch Gateway
joch Memory
joch Registry
joch Operator
joch Console
joch Cloud
joch Runtime
joch Specs
```

CLI examples:

```bash
joch up
joch apply -f agent.yaml
joch get agents
joch get executions
joch run research-agent "Research agent orchestration"
joch trace exec-123
joch approvals ls
joch memories ls
joch registry search personalities
joch deploy research-agent --env prod
```

---

# Final recommendation

Rebrand to:

```text
joch
```

Create these repos:

```text
github.com/peasantsai/joch
github.com/peasantsai/joch-spec
github.com/peasantsai/joch-docs
github.com/peasantsai/joch-examples
github.com/peasantsai/joch-registry
github.com/peasantsai/joch-operator
github.com/peasantsai/joch-mcp
github.com/peasantsai/joch-js
github.com/peasantsai/joch-python
github.com/peasantsai/joch-ui
github.com/peasantsai/joch-adapters
github.com/peasantsai/joch-cloud
```

Start with a monorepo-heavy approach, then split when boundaries harden:

```text
Now:
  joch
  joch-spec
  joch-docs
  joch-examples

Later:
  operator
  mcp
  sdk repos
  ui
  adapters
  cloud
```

That gives Joch a serious brand, a clean repo system, and room to grow into a real ecosystem.
