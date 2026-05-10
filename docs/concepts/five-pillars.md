# Five Pillars

Joch is organized around five product pillars. Each pillar maps to one or more services in the [architecture](../architecture/index.md), one or more resource kinds in the [resource catalog](../specs/index.md), and one or more end-to-end [use cases](../use-cases/index.md).

## 1. Inventory

> Know every agent, model, tool, MCP server, memory, RAG source, deployment, and owner — and the relationships between them.

Inventory is the system of record. It survives changes in framework, vendor, and team owner because the records live in Joch, not inside an SDK.

Capabilities:

- Discover agents written with OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, CrewAI, or custom Python/TypeScript code.
- Persist a versioned record per agent, including its framework adapter, model policy, tool list, MCP servers, memory bindings, owner, and policies.
- Generate a per-agent [Agent Bill of Materials](../specs/kubernetes/agbom.md) (AgBOM) extending CycloneDX, SPDX, and SWID, on demand and on every change.
- Browse, query, and label agents through the CLI, API, and web console.

[Read the Inventory pillar in depth](../pillars/inventory.md)

## 2. Governance

> Policy-as-code for tool use, data access, approvals, model choice, memory writes, and budgets — enforced at the right boundary, in the same language, across every framework.

Governance is the policy enforcement point. Each SDK has its own ad-hoc guardrails. Joch defines a portable policy resource and enforces it once at the gateway boundary, so all agents — regardless of SDK — get the same controls.

Capabilities:

- Portable [`Policy`](../specs/kubernetes/policy.md) resource for model, tool, network, data, budget, and audit rules.
- [Tool gateway](../architecture/tool-gateway.md) and [MCP gateway](../architecture/mcp-gateway.md) enforcement, with `allow`, `deny`, and `modify` outcomes per the [AOS hook contract](../aos/hooks.md).
- [Approval workflow](../specs/kubernetes/approval.md) for risky tool calls.
- PII redaction, prompt-injection scanning of tool results, egress and data-residency policies.

[Read the Governance pillar in depth](../pillars/governance.md)

## 3. Portability

> Run the same agent record locally, in Docker, on Kubernetes, or with managed runtimes — and migrate conversations across providers without losing tools, memory, or artifacts.

Portability is what makes Joch survive a change of SDK, vendor, or runtime. The agent record, the conversation event log, the canonical message format, and the migration checkpoint are all owned by Joch.

Capabilities:

- One [agent record](../specs/kubernetes/agent.md) runs against any framework via [framework adapters](../architecture/framework-adapters.md).
- Vendor-neutral [`Conversation`](../specs/kubernetes/conversation.md) and [`StateCheckpoint`](../specs/kubernetes/state-checkpoint.md) resources.
- Deterministic [migration checkpoints](../architecture/state-portability.md) for switching providers mid-conversation.
- [`ModelRoute`](../specs/kubernetes/model-route.md) for capability-aware, cost-aware, region-aware fallback across providers.

[Read the Portability pillar in depth](../pillars/portability.md)

## 4. Observability

> Trace every model call, tool call, memory write, RAG retrieval, approval, cost line item, and artifact — and surface the operator views that vendor SDKs do not.

Observability is the foundation that powers governance, release management, cost control, and incident response.

Capabilities:

- [`Trace`](../specs/kubernetes/trace.md) resource with sampling, retention, and OpenTelemetry / OCSF export, aligned with [AOS Trace events](../aos/events.md).
- Operator views: top agents, top tools, top models, cost by team, drift detection, policy denial heatmap, regression after deployment.
- Per-execution audit trail joining inputs, retrieved knowledge, model calls, tool calls, memory writes, costs, and outputs.
- Alerts on cost spikes, tool failure rate, model fallback rate, RAG retrieval quality, and guardrail violations.

[Read the Observability pillar in depth](../pillars/observability.md)

## 5. Release Management

> Version, eval, diff, promote, roll back, and audit agents like production software, with quality gates that block regressions before they reach customers.

Release management is what turns ad-hoc prompt iteration into production engineering.

Capabilities:

- Versioned agent records with diffable specs.
- [`Eval`](../specs/kubernetes/eval.md) resource with deterministic and LLM-judge metrics, schedules, and thresholds.
- Promotion across [`Environment`](../specs/kubernetes/environment.md) (e.g., `staging` → `prod`) with quality gates.
- One-command rollback by record version, with a full audit trail.
- CI/CD friendly: `joch validate -f .`, `joch diff`, `joch eval run`, `joch promote`.

[Read the Release Management pillar in depth](../pillars/release-management.md)

## How the pillars compose

The pillars are not independent — they reinforce one another:

```text
Inventory                   ──▶ enables Governance     (you cannot enforce policy on what you cannot see)
Governance + Inventory      ──▶ enable Portability     (portable policy lets you migrate safely)
Inventory + Portability     ──▶ enable Observability   (cross-SDK records create cross-SDK traces)
Observability               ──▶ enables Release Mgmt   (you cannot gate what you cannot measure)
Release Mgmt + Governance   ──▶ enable Compliance      (versioned, audited, policy-bound runs)
```

This composition is the basis of the [moat](../business/moat.md) — once an organization stores its agent inventory, policies, traces, evals, and approvals in Joch, swapping Joch out becomes harder than swapping any single SDK underneath it.
