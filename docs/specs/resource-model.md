# Resource Model

## Control-Plane Premise

I would solve OpenAI-to-Claude mid-conversation switching with:

```text
Canonical state store
+ event-sourced conversation log
+ normalized tool-call records
+ externalized memory/artifacts
+ migration checkpoints
+ provider adapters
+ capability validation
```

The most important architectural decision is this:

> `joch` should own agent identity, memory, and lifecycle.
> Vendors should only provide inference backends.

That is the difference between a real fleet manager and a thin wrapper around SDKs.

I would make `joch` specs feel like **Kubernetes CRDs for agent systems**.

The core idea:

```yaml
apiVersion: joch.dev/v1alpha1
kind: <ResourceKind>
metadata:
  name: example-resource
  labels:
    team: string
    env: string
spec:
  ...
status:
  ...
```

Every resource should have:

```yaml
apiVersion: joch.dev/v1alpha1
kind: <ResourceKind>
metadata:
  name: string
  namespace: string
  labels: {}
  annotations: {}
spec: {}
status:
  phase: Pending | Ready | Running | Failed | Suspended
  conditions: []
  observedGeneration: number
```

This gives Joch a common control-plane model for agents, models, memories, tools, deployments, and runtime state.

MCP should strongly influence the design: its official primitives include **tools**, **resources**, and **prompts**, with tools exposing callable functions and resources exposing context/data by URI. ([Model Context Protocol][1]) OpenAI’s Agents SDK also treats tools, handoffs, guardrails, tracing, and sessions as first-class orchestration concepts, which map well into `joch` resources. ([OpenAI GitHub][2])

---

## Top-Level Resource Kinds

I would start with these:

```text
Agent
Model
Skill
Tool
MCPServer
Personality
Memory
RAG
KnowledgeSource
Prompt
Plan
Execution
Deployment
Policy
Guardrail
Secret
Budget
Eval
Trace
Artifact
Environment
Team
```

The minimum useful v1 should include:

```text
Agent
Model
Skill
MCPServer
Personality
Memory
RAG
Plan
Execution
Deployment
Policy
Trace
```

The rest can come later.

## Kubernetes Specs

Concrete Kubernetes-style YAML contracts now live in the dedicated [Kubernetes Specs](kubernetes/index.md) section. Each resource kind has its own page under `docs/specs/kubernetes/`, keeping this overview focused on the shared model, relationships, API grouping, and design philosophy.

---

## Resource relationships

The graph should look like this:

```text
Deployment
  └── Agent
        ├── Model
        ├── Personality
        ├── Prompt
        ├── Skills
        │     └── Tools
        │           └── MCPServer
        ├── Memories
        ├── RAG
        │     └── KnowledgeSources
        ├── ExecutionLoop
        ├── Policies
        ├── Guardrails
        └── Budgets

Execution
  ├── Agent
  ├── Plan
  ├── Trace
  ├── ToolCalls
  ├── MemoryWrites
  └── Artifacts
```

This separation is important because different resources change at different speeds:

```text
Model: changes often
Personality: changes carefully
Policy: changes through governance
Memory: changes continuously
Execution: created per run
Deployment: changes through rollout
```

---

---

## Suggested v1alpha1 API groups

I would organize the API into groups:

```text
joch.dev/v1alpha1
  Agent
  Personality
  Prompt
  Skill

runtime.joch.dev/v1alpha1
  Execution
  Plan
  ExecutionLoop
  ToolCall
  Handoff
  Approval
  Conversation
  StateCheckpoint

model.joch.dev/v1alpha1
  Model
  ModelRoute
  ModelPolicy

tools.joch.dev/v1alpha1
  Tool
  MCPServer
  ToolRegistry

memory.joch.dev/v1alpha1
  Memory
  RAG
  KnowledgeSource

ops.joch.dev/v1alpha1
  Deployment
  Environment
  Budget
  Trace
  Eval
  Artifact
  Policy
  Guardrail
```

For a CLI, the user would still see simple commands:

```bash
joch get agents
joch get models
joch get skills
joch get memories
joch get rags
joch get executions
joch get deployments
```

---

---

## Additional resources

The obvious resources are already listed. I would add these:

```text
Prompt
Conversation
StateCheckpoint
Policy
Guardrail
Budget
Trace
Artifact
Approval
Handoff
KnowledgeSource
Eval
Environment
SecretRef
ModelRoute
ToolCall
```

The most important missed ones are:

```text
Conversation
StateCheckpoint
Policy
Guardrail
Trace
Approval
ToolCall
Artifact
```

Without those, we cannot get serious enterprise governance, reproducibility, or vendor migration.

---

---

## Example complete app bundle

A real project might look like:

```text
joch.yaml
agents/
  research-agent.yaml
  writer-agent.yaml
models/
  openai-gpt.yaml
  anthropic-claude.yaml
personalities/
  pragmatic-researcher.yaml
  technical-writer.yaml
prompts/
  research-system.yaml
skills/
  web-research.yaml
  report-writing.yaml
tools/
  web-search.yaml
  github-tools.yaml
mcp/
  github.yaml
  slack.yaml
memory/
  research-memory.yaml
rag/
  company-docs-rag.yaml
policies/
  default-policy.yaml
  pii-policy.yaml
guardrails/
  citation-required.yaml
deployments/
  research-agent-prod.yaml
evals/
  research-agent-eval.yaml
```

Then:

```bash
joch apply -f .
joch get agents
joch run research-agent -f tasks/market-research.yaml
joch logs execution/exec-123
joch trace execution/exec-123
joch approvals ls
```

---

---

## The north-star spec model

The cleanest mental model is:

```text
Desired state:
  Agent, Model, Skill, Personality, Memory, RAG, Policy, Deployment

Runtime state:
  Execution, Plan, ToolCall, Handoff, Approval, Trace, Artifact

Portable state:
  Conversation, StateCheckpoint, Memory, Artifact

Integration state:
  Tool, MCPServer, KnowledgeSource, SecretRef
```

That gives `joch` a real control-plane architecture.

The core design philosophy should be:

> Agents are not prompts.
> Agents are deployable, observable, governed, stateful resources.

That is the difference between a CLI wrapper and a true agent fleet manager.

[1]: https://modelcontextprotocol.io/specification/2025-11-25?utm_source=chatgpt.com "Specification"
[2]: https://openai.github.io/openai-agents-python/tracing/?utm_source=chatgpt.com "Tracing - OpenAI Agents SDK"
[3]: https://modelcontextprotocol.io/specification/2025-11-25/server/tools?utm_source=chatgpt.com "Tools"
[4]: https://www.tomshardware.com/tech-industry/artificial-intelligence/anthropics-model-context-protocol-has-critical-security-flaw-exposed?utm_source=chatgpt.com "Anthropic's Model Context Protocol includes a critical remote code execution vulnerability - newly discovered exploit puts 200,000 AI servers at risk"

[1]: https://modelcontextprotocol.io/specification/2025-11-25?utm_source=chatgpt.com "Specification"
[2]: https://openai.github.io/openai-agents-python/tracing/?utm_source=chatgpt.com "Tracing - OpenAI Agents SDK"
[3]: https://modelcontextprotocol.io/specification/2025-11-25/server/tools?utm_source=chatgpt.com "Tools"
[4]: https://www.tomshardware.com/tech-industry/artificial-intelligence/anthropics-model-context-protocol-has-critical-security-flaw-exposed?utm_source=chatgpt.com "Anthropic's Model Context Protocol includes a critical remote code execution vulnerability - newly discovered exploit puts 200,000 AI servers at risk"
