---
title: Joch AI - harness autonomous agents into accountable workforces
---

# Joch AI

> **Harness autonomous agents into accountable workforces.**

Joch AI is the open control plane for discovering, governing, securing, and auditing AI agents across frameworks, model providers, tools, and runtimes.

The MVP is intentionally narrow:

> **Secure MCP and tool governance, cross-framework agent inventory, and auditability.**

Joch AI is not another agent framework. Agent SDKs already provide authoring, orchestration, memory, tool calling, tracing, and deployment surfaces. Joch AI is the layer around those systems: the system of record, policy enforcement point, approval workflow, and audit log for agent action.

## Why Joch AI exists

Production agents are leaving demos and entering operational systems. They call tools, read data, write to SaaS products, run MCP servers, spend money, and change state. Most organizations do not have one place to answer:

```text
What agents exist?
Who owns them?
Which tools can they call?
Which MCP servers are running?
Which actions require approval?
What did each agent do?
What did each action cost?
Can we prove what happened later?
```

Joch AI makes autonomous agents visible, governed, portable, and accountable.

## The MVP wedge

The first product is **Joch Gateway**:

```text
Agent
  -> Joch Gateway
  -> Policy check
  -> Approval if required
  -> MCP server / API / tool
  -> Audit log
  -> Agent
```

Every agent framework needs tools. MCP adoption creates tool sprawl. Tool use creates security and compliance risk. Joch AI gives teams one audit and policy layer without forcing them to rewrite agents.

## What Joch AI gives teams

<div class="grid cards" markdown>

-   :material-format-list-bulleted-square: **Inventory**

    ---

    Discover and register agents, tools, MCP servers, owners, policies, risk levels, and last execution state.

    [Read the MVP scope](mvp/scope.md)

-   :material-shield-key-outline: **Governance**

    ---

    Enforce policy-as-code before risky tool calls, including external writes, code execution, financial actions, and privileged operations.

    [Read policy and approvals](architecture/policy-approvals.md)

-   :material-server-security: **MCP Gateway**

    ---

    Scan MCP servers, detect changed tools, classify side effects, proxy calls, and quarantine risky capabilities.

    [Read the gateway architecture](architecture/gateway.md)

-   :material-clipboard-check-outline: **Approvals**

    ---

    Pause external writes and other risky calls until a human approves or rejects them.

    [Read the approval resource](resources/approval.md)

-   :material-text-box-search-outline: **Audit**

    ---

    Persist tool calls, decisions, approvals, execution events, and ABOMs as durable records.

    [Read the data model](architecture/data-model.md)

-   :material-package-variant-closed: **ABOM**

    ---

    Generate an Agent Bill of Materials covering models, tools, MCP servers, memories, RAG sources, secrets referenced, policies, risk flags, and deployments.

    [Read the ABOM spec](resources/abom.md)

</div>

## Where to go next

- [Get started](getting-started/index.md) with a local install and first governed agent.
- [Read the positioning](strategy/positioning.md) to understand what Joch AI is and is not.
- [Review the MVP roadmap](mvp/roadmap.md) for the phased delivery plan.
- [Browse the resource model](resources/index.md) for the YAML API surface.

## Final strategic sentence

> **Joch AI wins by becoming the neutral control layer for agent action: the place where every agent, tool, policy, approval, and audit record comes together.**
