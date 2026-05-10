# Positioning

## What Joch AI is

> **Joch AI is the system of record, policy enforcement layer, and tool-governance gateway for agent fleets built anywhere.**

It surrounds existing agent frameworks instead of replacing them.

## What Joch AI is not

Joch AI should not try to be:

```text
another agent SDK
another workflow graph framework
another crew/task abstraction
another RAG framework
another model SDK
another proprietary tool-calling system
```

Those spaces are already crowded. The MVP stays focused on the layer around agent systems:

```text
agent inventory
MCP/tool governance
policy enforcement
ABOM generation
tool-call audit
cross-framework observability
approval workflows
portable deployment metadata
```

## Vendor SDKs build agents

OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, and CrewAI already cover agent authoring, tool calling, state, orchestration, and deployment surfaces.

Joch AI answers different questions:

```text
What agents exist across my company?
Which tools can they call?
Which agents can access customer data?
Which MCP servers are running?
Can I enforce one policy across frameworks?
Can I require approval before external writes?
Can I audit every tool call?
Can I generate an Agent Bill of Materials?
```

## Product test

Would a team using an existing SDK install Joch AI to write agents?

```text
No.
```

Would they install Joch AI to secure MCP tools, govern tool calls, track cost, approve risky actions, catalog agents, and audit activity?

```text
Yes.
```

That operator outcome is the product.
