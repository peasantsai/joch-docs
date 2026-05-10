# Moat

Joch's defensibility does not come from a clever algorithm. It comes from owning the cross-cutting state and policy boundary that every SDK eventually crosses, and from the durability of the records that accrue once Joch is in place.

## The moat sources

### 1. Cross-framework metadata graph

Once Joch holds the inventory of every agent across OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, CrewAI, and custom code — plus the relationships between agents, tools, models, MCP servers, memory, and policies — it becomes the system of record for agent operations. Replacing it is harder than replacing any single SDK underneath it.

### 2. Policy enforcement at the boundary

Each SDK does guardrails differently. Joch's [Policy](../specs/kubernetes/policy.md) is a single resource enforced at the tool, model, memory, and MCP boundary that every SDK crosses. The customer writes the rule once and the rule applies fleet-wide. Migrating off Joch means re-implementing the same rules per-SDK and losing portability.

### 3. Portable execution and state model

[`Conversation`](../specs/kubernetes/conversation.md) and [`StateCheckpoint`](../specs/kubernetes/state-checkpoint.md) are vendor-neutral. Once a customer's conversation history lives in Joch, switching to another control plane requires re-deriving canonical state from raw provider transcripts — a lossy operation.

### 4. Historical traces, evals, and approvals

Trace history, eval results, approval decisions, and ABOM diffs only grow more valuable over time. They are the substrate of audit, regression analysis, and compliance. The longer Joch is in place, the more switching cost the customer takes on by moving away.

### 5. Trusted registry

The [registry](#) of vetted MCP servers, policies, agent templates, and tool definitions becomes a network effect: more contributors → better catalog → more adoption → more contributors. CycloneDX/SPDX/SWID signing through Joch ABOM amplifies the trust signal further.

### 6. Enterprise deployment integrations

Approval routing into Slack and email; trace export into existing OTel and SIEM stacks; secret resolution into Vault, Kubernetes, AWS Secrets Manager; runtime adapters for Local / Docker / Kubernetes / Nomad. Each integration is a small switching cost on its own; together, they compose into a meaningful one.

### 7. AOS conformance

Implementing [OWASP AOS](../aos/index.md) faithfully — AgBOM, hooks, trace events, OpenTelemetry, OCSF — turns a product feature into a standards alignment. Customers buying for compliance do not need bespoke integrations.

## Brutal honesty: what is *not* a moat

- **Code.** Joch's code is open source. Forks are allowed by license.
- **A clever single feature.** Any single feature can be cloned in a quarter.
- **Vendor lock-in.** We deliberately avoid it; portability is the value proposition.

The moat is the *combined* presence of inventory + policy + state + history + integrations + standards conformance, in one product, kept current as the SDK ecosystem evolves.

## Why the category is durable

The agent SDK market is a multi-vendor land grab. Each vendor will keep adding capabilities. Each vendor's capabilities will be incompatible with the others' in subtle ways. Operators will keep needing a layer above:

- to know what they have,
- to enforce policy uniformly,
- to migrate when economics change,
- to audit when regulators ask,
- to control cost when bills surprise.

That layer is the control plane. Whoever owns it for an organization is hard to remove because the records, policies, and history are owned together.

## Defensive posture

We minimize risk by:

- **Open sourcing the core generously.** Forking is allowed, but the activity-and-network advantage stays with the canonical project.
- **Aligning to AOS and adjacent standards** so customers do not feel locked in to Joch's primitives.
- **Keeping the data plane small.** A small, auditable boundary is easier to defend than a sprawling one.
- **Investing in framework adapters first**, so each SDK release strengthens us instead of weakening us.

The moat compounds. The first customer is the hardest. The hundredth is much easier because the AOS exports, registry contributions, and operator views by then describe a category, not a product.
