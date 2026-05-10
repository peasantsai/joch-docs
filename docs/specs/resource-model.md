# Resource Model

Joch's resource model is a Kubernetes-style API surface for **operating** AI agent fleets. It deliberately stops short of authoring agents — that work belongs in the SDK each agent was built with. The kinds in the catalog represent the records, decisions, runtime artifacts, and operational rules that a control plane needs.

## Premise

> Joch owns agent identity, governance, runtime state, and audit.
> Vendor SDKs own the agent loop, prompts, and tool implementation.
> Vendor providers own model inference.

This is the difference between a real fleet manager and a thin wrapper around SDKs.

## Top-level kinds

```text
Identity / desired state                 Runtime                Operations
─────────────────────────────────        ────────────           ─────────────
Agent                                    Execution              Deployment
FrameworkAdapter                         Conversation           Environment
Model                                    StateCheckpoint        Team / Namespace
ModelRoute                               ToolCall               Budget
Tool                                     Approval               Eval
MCPServer                                Handoff                Secret
Memory                                   Trace                  ABOM
RAG                                      Artifact
KnowledgeSource
Policy
```

The minimum useful v1 set:

```text
Agent, FrameworkAdapter, Model, ModelRoute, Tool, MCPServer,
Policy, Approval, ToolCall, Trace, Conversation, StateCheckpoint,
Memory, RAG, ABOM, Execution, Deployment, Eval, Environment, Team / Namespace
```

The rest can land later without breaking schemas.

## What is *not* a resource kind

Joch deliberately excludes kinds that belong inside the SDK:

```text
Personality                in the SDK / agent code
Skill                      in the SDK / agent code (composes tools, prompts, planners)
Prompt                     in the SDK / agent code
Plan                       in the SDK at runtime, summarized into a Trace event
ExecutionLoop              in the SDK
Guardrail                  replaced by Joch Policy + AOS hooks at gateway boundaries
```

Trying to model these as control-plane resources would force every SDK into a single shape and pull Joch into the SDK competition. Joch governs the **boundary** SDKs cross instead.

## Common envelope

Every resource shares a Kubernetes-style envelope:

```yaml
apiVersion: joch.dev/v1alpha1
kind: <Kind>
metadata:
  name: string
  namespace: string
  labels:
    team: string
    env: dev | staging | prod
  annotations: {}
spec: {}
status:
  phase: Pending | Ready | Running | Failed | Suspended
  conditions: []
  observedGeneration: number
```

The shared envelope makes the API discoverable, cacheable, and CLI-friendly across the catalog.

## API groups

```text
joch.dev/v1alpha1
  Agent
  FrameworkAdapter
  Policy

runtime.joch.dev/v1alpha1
  Execution
  ToolCall
  Handoff
  Approval
  Conversation
  StateCheckpoint

model.joch.dev/v1alpha1
  Model
  ModelRoute

tools.joch.dev/v1alpha1
  Tool
  MCPServer

memory.joch.dev/v1alpha1
  Memory
  RAG
  KnowledgeSource

ops.joch.dev/v1alpha1
  Deployment
  Environment
  Team
  Budget
  Trace
  Eval
  Artifact
  ABOM
  Secret
```

Operators rarely need to think about API groups; the CLI is `joch get <kind>` and the kind is unambiguous.

## Relationships

```text
Deployment
  └── Agent
        ├── FrameworkAdapter
        ├── ModelRoute ──┐
        ├── Tools ───────┼── ToolCall (runtime)
        ├── MCPServers ──┘
        ├── Memories
        ├── RAG
        │     └── KnowledgeSources
        ├── Policies
        ├── Budgets
        └── ABOM (derived)

Execution
  ├── Agent
  ├── Conversation
  │     └── StateCheckpoint
  ├── Trace
  ├── ToolCalls
  ├── Approvals
  ├── Handoffs
  └── Artifacts
```

Resources move at different speeds. `Model` and `ModelRoute` change often. `Personality` (in the SDK) and `Policy` change carefully. `Memory` changes continuously. `Execution` is created per run. `Deployment` changes through controlled rollouts.

## Project layout convention

A real project ships its records as a directory:

```text
joch.yaml
agents/
  support-triage.yaml
framework-adapters/
  openai-agents-sdk.yaml
  claude-agent-sdk.yaml
models/
  gpt-5-thinking.yaml
  claude-sonnet.yaml
model-routes/
  research-default.yaml
tools/
  zendesk-search.yaml
  slack-send.yaml
mcp-servers/
  github.yaml
memory/
  support-working-memory.yaml
rag/
  support-docs-rag.yaml
policies/
  no-external-send-without-approval.yaml
deployments/
  support-triage-prod.yaml
environments/
  prod.yaml
evals/
  support-triage-quality.yaml
budgets/
  support-platform-monthly.yaml
```

```bash
joch apply -f .
joch get agents
joch run support-triage -f cases/triage.yaml
joch trace last
joch approvals ls
```

## North-star summary

```text
Desired state    Agent, FrameworkAdapter, Model, ModelRoute, Tool, MCPServer,
                 Memory, RAG, KnowledgeSource, Policy, Deployment, Environment

Runtime state    Execution, Conversation, StateCheckpoint, ToolCall, Handoff,
                 Approval, Trace, Artifact

Operations       Budget, Eval, ABOM, Team, Secret
```

The design philosophy:

> Agents are not prompts. Agents are deployable, observable, governed, stateful records with vendored implementations.

That sentence is what separates a control plane from a CLI wrapper.
