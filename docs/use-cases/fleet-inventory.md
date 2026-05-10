# Fleet Inventory

> A platform team needs to know every agent the company runs, regardless of who built it or which SDK they used, plus which models, tools, MCP servers, and data sources each one depends on.

## Symptoms

- A vendor announces a price change and nobody knows how many agents would be affected.
- A security review asks "which agents can call `email.send`?" and there is no central answer.
- Cost reports break down by provider, but not by team or by agent.
- Six teams build agents in three different SDKs and there is no shared list of what exists.

## What Joch does

Joch [Inventory](../pillars/inventory.md) keeps a versioned record per agent, populated by [framework adapter discovery](../architecture/framework-adapters.md) and consumable through the CLI, API, and console.

## Walkthrough

### 1. Discover what exists

```bash
joch discover --framework openai-agents-sdk --path ./services
joch discover --framework claude-agent-sdk    --path ./coding-agents
joch discover --framework langgraph           --path ./pipelines
joch discover --framework crewai              --path ./marketing
```

Each discovery call produces stub `Agent` records with framework metadata, detected tools, and detected MCP servers.

### 2. Review and apply

```bash
joch get agents --pending
joch describe agent stub/support-triage
joch apply -f agents/support-triage.yaml
```

Owners fill in policies, model routes, and budgets before applying.

### 3. Browse the fleet

```bash
joch get agents
joch get agents --owner support-platform --env prod
joch get agents --framework claude-agent-sdk
joch get tools --agent support-triage
joch get mcpservers --in-use
joch get models --in-use
```

### 4. Generate ABOM

```bash
joch abom support-triage
joch abom support-triage --format cyclonedx > support-triage.cdx.json
joch abom ls --high-risk
```

The [ABOM](../specs/kubernetes/abom.md) is the inventory's audit-grade output: every dependency, with version and owner.

### 5. Answer the hard questions

```bash
joch get agents --tool-can-call email.send
joch get agents --reads-classification customer-tier
joch get agents --using-model openai:gpt-5-thinking
joch get agents --has-mcp-server github --pinned-version 1.2.0
joch get agents --not-evaluated-since 30d
```

Each query joins the inventory graph: agent → tool, agent → memory, agent → MCP server, agent → eval. The answers are deterministic.

## Resources involved

- [`Agent`](../specs/kubernetes/agent.md), [`FrameworkAdapter`](../specs/kubernetes/framework-adapter.md)
- [`Tool`](../specs/kubernetes/tool.md), [`MCPServer`](../specs/kubernetes/mcpserver.md)
- [`Model`](../specs/kubernetes/model.md), [`ModelRoute`](../specs/kubernetes/model-route.md)
- [`Memory`](../specs/kubernetes/memory.md), [`RAG`](../specs/kubernetes/rag.md), [`KnowledgeSource`](../specs/kubernetes/knowledge-source.md)
- [`ABOM`](../specs/kubernetes/abom.md)
- [`Team / Namespace`](../specs/kubernetes/team-namespace.md), [`Environment`](../specs/kubernetes/environment.md)

## Outcome

The platform team can answer ownership, dependency, and capability questions without writing custom queries. Inventory is the foundation that makes governance, cost control, and release management possible.
