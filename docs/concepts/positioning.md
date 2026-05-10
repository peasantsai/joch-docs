# Positioning

## The category Joch is in

> **Joch is the portable control plane for AI agent fleets.**

Or, more specifically:

> **Joch manages the lifecycle, governance, observability, portability, and deployment of agents built with any SDK.**

Joch is in the same category as Kubernetes (control plane for containers), Terraform (control plane for cloud resources), and Backstage (control plane for services and ownership) — adapted to the operational reality of AI agents.

## The category Joch is not in

Joch is **not**:

- another agent framework
- a better OpenAI Agents SDK
- a better Claude Agent SDK
- a better Google ADK or Microsoft Agent Framework
- a better LangGraph or CrewAI
- a hosted notebook or playground
- a model router by itself
- an MCP server by itself
- a benchmarking suite by itself

Each of those is a single capability that vendor SDKs and specialized tools already cover. Joch composes those capabilities into a coherent **operations layer** that survives a change of SDK, vendor, or runtime.

## What vendor SDKs already do

| Platform | Already provides |
|---|---|
| **OpenAI Agents SDK** | Code-first agents, planning, tool use, collaboration, state, guardrails, human review, MCP/tools, tracing surfaces. ([docs](https://developers.openai.com/api/docs/guides/agents)) |
| **Claude Agent SDK** | Production agents with Claude Code as a library: file access, shell commands, web search, editing code, tools, hooks, subagents, MCP, permissions, sessions. ([docs](https://code.claude.com/docs/en/agent-sdk/overview)) |
| **Google ADK** | Open-source framework to build, debug, deploy, evaluate, and scale agents; tools, multi-agent systems, workflow agents, dynamic routing. ([docs](https://docs.cloud.google.com/gemini-enterprise-agent-platform/build/adk)) |
| **Microsoft Agent Framework** | Multi-agent orchestration, workflows, MCP, A2A, OpenAPI integration, observability, durability, compliance, Azure AI Foundry deployment and governance. ([blog](https://azure.microsoft.com/en-us/blog/introducing-microsoft-agent-framework/)) |

Joch does not compete with these. Joch wraps and governs them.

## The two questions

Vendor SDKs answer:

```text
How do I build an agent?
How does the agent loop work?
How do I call tools?
How do I use this provider's models?
How do I run this agent in this ecosystem?
```

Joch answers:

```text
What agents exist across my company?
Who owns them?
What models, tools, MCP servers, and data do they touch?
What did they do yesterday, and what did it cost?
Which ones are broken, expensive, or out of policy?
Can I migrate an agent from one provider to another safely?
Can I block tool calls without approval, fleet-wide?
Can I deploy the same spec locally, in Docker, and in Kubernetes?
Can I version, eval, diff, promote, and roll back agents?
Can I audit every tool call, memory write, and policy denial?
```

The first set is a developer problem. The second set is an operator problem. Joch is built for the second.

## What Joch will deliberately not build first

To stay sharp and avoid becoming a weak clone of an SDK, Joch will **not** build, in v1:

- a proprietary agent loop competing with every SDK
- a proprietary tool-calling abstraction that ignores MCP
- a model SDK
- a workflow DSL before integrations
- a RAG framework as the main value prop
- a hosted sandbox as the first feature

These spaces are crowded and expensive. Joch will instead **wrap and govern** existing agent systems and bring the controls that enterprises expect.

## Where the value compounds

Once Joch holds:

- the cross-framework agent inventory,
- the policy enforcement point at the tool, model, and memory boundary,
- the portable execution and state model,
- the historical traces, evals, costs, and approvals,
- the registry of trusted tools, skills, policies, and templates,
- and the deployment integrations,

it becomes the **system of record** for how an organization operates AI agents. That is the long-term value Joch is built to capture.

## Brutally honest product test

Ask:

```text
Would a team using OpenAI Agents SDK install Joch to write agents?      No.
Would they install Joch to secure MCP tools, govern tool calls,
  track costs, standardize deployment, audit executions, manage
  approvals, compare versions, enforce policies, catalog agents,
  and migrate models?                                                    Yes.
```

Those operator outcomes are what Joch is built to deliver. The rest of this documentation describes how.
