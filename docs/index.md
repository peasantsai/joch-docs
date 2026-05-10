---
title: Joch — the vendor-neutral control plane for AI agent fleets
---

# Joch

> **Joch is the portable control plane for AI agent fleets.**
> It manages the inventory, governance, portability, observability, and release lifecycle of agents built with **any** SDK — OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, CrewAI, or your own Python and TypeScript code.

Joch is **not** another agent SDK, framework, or runtime. The vendor SDKs already cover code-first agents, planning, tool use, collaboration, state, guardrails, and human review. Joch sits **above** those SDKs and answers a different question: *how do you operate, secure, and evolve a fleet of agents across vendors and frameworks?*

## What Joch is for

Joch answers operational questions that vendor SDKs leave to the operator:

```text
What agents exist across my company, and who owns them?
What models, tools, MCP servers, and memory does each agent depend on?
What did each agent do yesterday, and what did it cost?
Which agents are using Claude, OpenAI, Gemini, or local models?
Can I move an agent from OpenAI to Claude — and what would break?
Can I block all agents from calling email.send without human approval?
Can I deploy the same agent locally, in Docker, and in Kubernetes?
Can I compare two agent versions before promoting to production?
Can I audit every tool call, memory write, and policy denial?
Which MCP servers are pinned, and which were quarantined this week?
```

If you are writing an agent today, you do not need Joch. If you are operating ten, a hundred, or a thousand of them — across teams, vendors, and environments — you do.

## The five pillars

<div class="grid cards" markdown>

-   :material-format-list-bulleted-square: **Inventory**

    ---

    A single system of record for every agent, model, tool, MCP server, memory, RAG source, deployment, and owner — regardless of which SDK built them.

    [Read the Inventory pillar](pillars/inventory.md)

-   :material-shield-key-outline: **Governance**

    ---

    Portable policy-as-code for tool use, data access, approvals, model choice, memory writes, and budgets — enforced uniformly across SDKs.

    [Read the Governance pillar](pillars/governance.md)

-   :material-swap-horizontal-bold: **Portability**

    ---

    One agent record runs locally, in Docker, on Kubernetes, or via managed runtimes; conversation state migrates across providers without losing tools, memory, or artifacts.

    [Read the Portability pillar](pillars/portability.md)

-   :material-radar: **Observability**

    ---

    Every model call, tool call, memory write, RAG retrieval, approval, cost line item, and artifact is captured as an event — exported to OpenTelemetry, OCSF, and your existing stack.

    [Read the Observability pillar](pillars/observability.md)

-   :material-source-branch-check: **Release Management**

    ---

    Version, eval, diff, promote, roll back, and audit agents like production software, with quality gates that block regressions before they reach customers.

    [Read the Release Management pillar](pillars/release-management.md)

</div>

## The strongest wedge

> **Joch is the secure MCP and tool-call gateway with an agent inventory layer.**

Every major SDK is adopting [Model Context Protocol (MCP)](https://modelcontextprotocol.io) and tool calling. That creates a new enterprise problem — untrusted servers, unknown capabilities, no central approval, no shared audit trail, no schema-drift detection, no side-effect controls, no portable policy. Joch makes the tool boundary safe, then expands outward into full agent fleet management.

[Explore the MCP gateway design](architecture/mcp-gateway.md) · [Explore the tool gateway design](architecture/tool-gateway.md)

## OWASP AOS conformance

Joch implements the **[OWASP Agent Observability Standard](https://aos.owasp.org/)** as its conformance baseline:

- **Inspect** — every Joch agent emits an Agent Bill of Materials ([AgBOM](aos/agbom.md)) extending CycloneDX, SPDX, and SWID.
- **Instrument** — Joch exposes the AOS [hooks](aos/hooks.md) (`agentTrigger`, `toolCallRequest`, `toolCallResult`, `message`, `memoryContextRetrieval`, `memoryStore`, `knowledgeRetrieval`, MCP, A2A) so any Guardian Agent can `allow`, `deny`, or `modify` agent decisions.
- **Trace** — Joch trace [events](aos/events.md) extend OpenTelemetry and OCSF schemas.

[Read the AOS conformance index](aos/index.md)

## Where to go next

<div class="grid cards" markdown>

-   :material-compass-outline: **Concepts**

    ---

    The product positioning, the five pillars in depth, the comparison against vendor SDKs, and a shared glossary.

    [Start with concepts](concepts/index.md)

-   :material-cog-outline: **Architecture**

    ---

    The control plane, data plane, framework adapters, tool and MCP gateways, policy engine, model router, and trust model.

    [Read the architecture](architecture/index.md)

-   :material-shape-outline: **Resources**

    ---

    Kubernetes-style YAML specifications for every resource kind in the Joch control plane.

    [Browse the resource catalog](specs/index.md)

-   :material-shield-check-outline: **AOS Conformance**

    ---

    How Joch implements the OWASP Agent Observability Standard for inspect, instrument, and trace.

    [Open the AOS section](aos/index.md)

-   :material-rocket-launch-outline: **Use Cases**

    ---

    End-to-end operator workflows: fleet inventory, MCP governance, cross-provider migration, cost control, release gates, approvals.

    [See the use cases](use-cases/index.md)

-   :material-briefcase-variant-outline: **Business**

    ---

    Product positioning, moat, competitive landscape, revenue models, target audience, go-to-market, open-core strategy, and roadmap.

    [Open the business section](business/index.md)

</div>

## Open source

Joch is released under the Apache-2.0 license by [PeasantsAI](https://github.com/peasantsai). The control-plane code, resource specs, AOS adapters, MCP gateway, and SDK adapters are all open. Hosted, multi-tenant features (`joch cloud`) are commercial.

[Read the contributing guide](community/contributing.md) · [Project governance](community/governance.md)
