# Comparison

Joch sits **above** vendor agent SDKs and frameworks, not next to them. The most common confusion is that Joch is "another way to define and run agents." It is not. The matrix below shows where vendor SDKs end and where Joch begins.

## Capability matrix

| Capability | OpenAI Agents SDK | Claude Agent SDK | Google ADK | MS Agent Framework | LangGraph | CrewAI | **Joch** |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| Author an agent in code | Yes | Yes | Yes | Yes | Yes | Yes | No |
| Model provider integrations | Own | Own | Own | Own | Multiple | Multiple | All (governs) |
| Tool calling / MCP | Yes | Yes | Yes | Yes | Partial | Partial | Yes (gateway) |
| Built-in agent loop / planner | Yes | Yes | Yes | Yes | Yes | Yes | No |
| Per-call tracing surface | Yes | Yes | Yes | Yes | Yes | Partial | Yes (cross-SDK) |
| Cross-SDK agent inventory | No | No | No | No | No | No | **Yes** |
| Portable policy-as-code | No | No | No | Partial | No | No | **Yes** |
| MCP / tool-call gateway | No | No | No | No | No | No | **Yes** |
| Cross-vendor model routing | No | No | No | Partial | Partial | No | **Yes** |
| Vendor-neutral state and migration | No | No | No | No | No | No | **Yes** |
| Agent Bill of Materials (AOS / CycloneDX) | No | No | No | No | No | No | **Yes** |
| Approval workflows | Partial | Partial | Partial | Partial | No | No | **Yes (portable)** |
| Eval and release gates | Partial | Partial | Partial | Partial | Partial | No | **Yes (portable)** |
| Multi-runtime deployment (Local / Docker / Kubernetes) | Partial | Partial | Partial | Yes | Partial | Partial | **Yes (one spec)** |
| Cost accounting across vendors | No | No | No | Partial | No | No | **Yes** |
| Centralized audit log of every decision | Partial | Partial | Partial | Partial | Partial | No | **Yes** |

"Yes" means a first-class, supported capability. "Partial" means available but not unified across all the framework's surfaces. "No" means out of scope for that framework.

## What changes when you adopt Joch

You **keep** the SDKs you already use:

- Teams that prefer OpenAI Agents SDK keep building OpenAI agents.
- Teams that prefer Claude Agent SDK keep building Claude agents.
- Teams using Google ADK, Microsoft Agent Framework, LangGraph, or CrewAI keep using them.
- Custom Python and TypeScript agents keep their existing code.

You **add** a control plane on top:

- Each agent gets registered in Joch via a [framework adapter](../architecture/framework-adapters.md).
- Tool calls flow through the [Joch tool gateway](../architecture/tool-gateway.md), enforcing the same [policies](../specs/kubernetes/policy.md) regardless of SDK.
- MCP servers register with the [MCP gateway](../architecture/mcp-gateway.md), where they are pinned, scanned, and version-tracked.
- All decisions emit [AOS-aligned trace events](../aos/events.md) and update the per-agent [ABOM](../specs/kubernetes/abom.md).

## Why this is not "yet another wrapper"

The control-plane category is durable because:

- **Coverage is the value.** A wrapper around one SDK is replaceable. A control plane that knows about every SDK in the org is not.
- **Policy is portable.** Each SDK does guardrails differently; Joch defines the rule once and enforces it at the boundary they all cross — the tool call, the model call, the memory write.
- **State outlives runtimes.** When a team migrates from OpenAI to Claude (or vice versa), Joch keeps the conversation, the memory references, and the artifact graph intact.
- **Compliance is cumulative.** ABOM, audit, traces, evals, approvals — they only get more valuable the longer they exist. The control plane is the only place where they can accrete.

For the underlying business case, see [moat](../business/moat.md) and [revenue models](../business/revenue-models.md).
