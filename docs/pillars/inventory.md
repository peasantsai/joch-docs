# Inventory

> Know every agent, model, tool, MCP server, memory, RAG source, deployment, and owner — and the relationships between them.

Inventory is the system of record. It is the foundation that the other four pillars depend on: you cannot govern, port, observe, or release what you cannot see.

## What Joch's inventory contains

Joch's inventory is a versioned graph of typed records. Every record uses Kubernetes-style YAML and is reachable through the [resource catalog](../specs/index.md).

```text
Agent              one record per logical agent, framework-agnostic
FrameworkAdapter   how this agent connects to its SDK runtime
Model              one record per model backend (provider + name + capabilities)
ModelRoute         capability-aware, cost-aware fallback policy
Tool               one record per callable function
MCPServer          one record per MCP server (with discovery + version pinning)
ToolCall           one record per concrete tool invocation
Policy             portable policy-as-code
ABOM               agent bill of materials extending CycloneDX/SPDX/SWID
Memory             working / semantic / episodic memory bindings
RAG                indices + their KnowledgeSources
Conversation       vendor-neutral dialog history
StateCheckpoint    portable mid-conversation snapshot
Trace              event log per execution
Eval               scored evaluations
Deployment         where and how many instances run
Approval           human review records
Budget             cost / usage caps
Environment        dev / staging / prod boundary
Team / Namespace   multi-tenant ownership boundary
Handoff            A2A transfer record
Artifact           durable execution outputs
Secret             external secret references
```

## How agents are registered

Agents are not authored in Joch. They are authored in OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, CrewAI, or custom code. Joch **registers** each one through a [`FrameworkAdapter`](../specs/kubernetes/framework-adapter.md):

```yaml
apiVersion: joch.dev/v1alpha1
kind: Agent
metadata:
  name: support-triage
  namespace: support-platform
  labels:
    owner: support-platform
    env: prod
spec:
  framework:
    adapterRef:
      name: openai-agents-sdk
    entrypoint: ./agents/support_triage.py
    pythonModule: support.agents.triage:agent
  modelPolicy:
    allowed:
      - openai:gpt-5-thinking
      - anthropic:claude-sonnet
  tools:
    - name: zendesk.search
    - name: slack.send
  mcpServers:
    - name: github
  policies:
    - name: no-customer-data-exfiltration
    - name: external-send-requires-approval
```

The `framework.adapterRef` is the only field that varies across SDKs. Everything else — model policy, tools, MCP servers, policies, budgets, observability — is shared across the fleet.

## Discovery

Joch ships with discovery commands that scan a repository or path for agent code in a given framework, register stubs, and let an owner finalize the records:

```bash
joch discover --framework openai-agents-sdk --path ./services
joch discover --framework claude-agent-sdk --path ./coding-agents
joch discover --framework langgraph --path ./pipelines
joch discover --framework crewai --path ./marketing
```

For frameworks that publish an agent manifest (e.g., A2A agent cards, Microsoft Agent Framework agent definitions), discovery reads the manifest directly.

## Operator commands

```bash
joch get agents
joch get agents --owner support-platform --env prod
joch describe agent support-triage
joch get tools --agent support-triage
joch get executions --agent support-triage --since 24h
joch abom support-triage
joch abom support-triage --format cyclonedx > support-triage.cdx.json
```

The CLI mirrors the resource catalog. The web console exposes the same data with filters, search, and ownership views.

## Why inventory comes first

Without inventory, every other pillar is best-effort:

- **Governance** without inventory enforces policy on the agents you remembered to enroll, and silently misses the ones you did not.
- **Portability** without inventory fragments state across SDKs because there is no canonical record to migrate.
- **Observability** without inventory produces orphaned traces with no owner, no version, and no policy context.
- **Release management** without inventory cannot version, diff, or roll back what it cannot identify.

The inventory pillar is also the prerequisite for compliance and audit: ABOM, ownership, and version history are inventory operations on top of the agent record.

## Acceptance criteria

A team operating Joch's Inventory pillar can answer the following without writing a query:

- How many agents do we run, by team, environment, and framework?
- Which agents can call `email.send`, `github.create_issue`, or `shell.exec`?
- Which agents read customer-tier data?
- Which MCP servers are unpinned, abandoned, or untrusted?
- Which agents are using the most expensive model in our route policy?
- Which agents have not been re-evaluated in the last 30 days?

If those answers are visible in `joch get …`, the Inventory pillar is doing its job.
