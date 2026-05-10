# Glossary

A single source of truth for the vocabulary used across Joch documentation. Where a term has a dedicated resource or page, the glossary entry links to it.

## A

**A2A (Agent-to-Agent)** — A protocol for inter-agent communication. Joch persists A2A interactions as `Handoff` events and exposes them through AOS-compliant hooks.

**AgBOM (Agent Bill of Materials)** — A machine-readable inventory of every component a Joch agent depends on: models, tools, MCP servers, knowledge sources, memory stores, policies, secrets, and deployments. Joch's AgBOM extends [OWASP AgBOM](https://aos.owasp.org/) and emits CycloneDX, SPDX, and SWID. See [`AgBOM`](../specs/kubernetes/agbom.md).

**Agent** (Joch resource) — The Joch record of an agent. The agent record is framework-agnostic; the actual agent code lives in OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, CrewAI, or custom code, and is connected via a [`FrameworkAdapter`](../specs/kubernetes/framework-adapter.md). See [`Agent`](../specs/kubernetes/agent.md).

**AgBOM** — See AgBOM.

**AOS (Agent Observability Standard)** — The OWASP standard Joch implements for inspect, instrument, and trace. See [AOS Conformance](../aos/index.md).

**Approval** — A policy-required human review of a side-effecting action (e.g., `email.send`, `github.create_issue`). See [`Approval`](../specs/kubernetes/approval.md).

**Artifact** — Any durable output of an execution (report, dataset, file, image). Stored by reference. See [`Artifact`](../specs/kubernetes/artifact.md).

## B

**Budget** — A cost or usage cap that the policy engine enforces before model calls, tool calls, or executions exceed it. See [`Budget`](../specs/kubernetes/budget.md).

## C

**Control plane** — The Joch services that own desired state, policy, inventory, approvals, and release gates. Counterpart to the data plane. See [Control Plane](../architecture/control-plane.md).

**Conversation** — A vendor-neutral, durable record of an agent's dialog. Survives provider migration. See [`Conversation`](../specs/kubernetes/conversation.md).

**CycloneDX** — An OWASP-related BOM standard supported by Joch's AgBOM emitter. See [CycloneDX Mapping](../aos/cyclonedx-mapping.md).

## D

**Data plane** — The Joch services that execute model calls, tool calls, memory reads/writes, RAG retrievals, and trace emission. Counterpart to the control plane. See [Data Plane](../architecture/data-plane.md).

**Deployment** — How many instances of an agent run, where, and at what scale. See [`Deployment`](../specs/kubernetes/deployment.md).

## E

**Environment** — A namespace + policy bundle that segments dev / staging / prod. See [`Environment`](../specs/kubernetes/environment.md).

**Eval** — A scored evaluation of an agent against a dataset, with metrics, thresholds, and an optional release gate. See [`Eval`](../specs/kubernetes/eval.md).

**Execution** — One concrete run of an agent. Owns model calls, tool calls, memory writes, traces, costs, and artifacts. See [`Execution`](../specs/kubernetes/execution.md).

## F

**FrameworkAdapter** — The Joch resource that connects an agent record to a specific SDK or framework runtime (OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, CrewAI, custom). See [`FrameworkAdapter`](../specs/kubernetes/framework-adapter.md).

## G

**Guardian Agent** — In OWASP AOS terminology, the policy enforcement entity that receives hook calls and returns `allow`, `deny`, or `modify`. In Joch, the Guardian Agent role is filled by the [policy engine](../architecture/policy-engine.md).

## H

**Handoff** — A transfer of control between agents (A2A). See [`Handoff`](../specs/kubernetes/handoff.md).

**Hook** — In AOS, a synchronous interception point in the agent loop (e.g., `agentTrigger`, `toolCallRequest`, `toolCallResult`, `message`, `memoryContextRetrieval`, `memoryStore`, `knowledgeRetrieval`). Joch implements all standard AOS hooks at its gateways. See [Hooks](../aos/hooks.md).

## I

**Inspect** — One of the three AOS pillars: agents publish a current AgBOM that auditors and runtime systems can fetch. Implemented in Joch by the AgBOM service.

**Instrument** — One of the three AOS pillars: agents expose hooks that a Guardian Agent can use to `allow`, `deny`, or `modify` decisions. Implemented in Joch by the policy engine and the tool / MCP gateways.

**Inventory (pillar)** — See [Inventory](../pillars/inventory.md).

## K

**KnowledgeSource** — A pointer to a corpus that feeds RAG indices (file, URL, S3, database). See [`KnowledgeSource`](../specs/kubernetes/knowledge-source.md).

## M

**MCP (Model Context Protocol)** — The protocol used by SDKs to expose tools, resources, and prompts to agents. See the [MCP gateway](../architecture/mcp-gateway.md) and [`MCPServer`](../specs/kubernetes/mcpserver.md).

**Memory** — A bound, durable scratchpad for an agent (working, semantic, episodic). See [`Memory`](../specs/kubernetes/memory.md).

**Model** — A model record describing a backend capability (provider, name, capabilities, limits, pricing). See [`Model`](../specs/kubernetes/model.md).

**ModelRoute** — A capability-aware, cost-aware routing policy for selecting and falling back across providers. See [`ModelRoute`](../specs/kubernetes/model-route.md).

## O

**OCSF** — Open Cybersecurity Schema Framework. Joch trace events extend the OCSF event taxonomy. See [OCSF Mapping](../aos/ocsf-mapping.md).

**OpenTelemetry** — The CNCF observability standard. Joch trace events extend OTel semantic conventions. See [OpenTelemetry Mapping](../aos/opentelemetry-mapping.md).

## P

**Policy** — A versioned, portable set of rules enforced by the policy engine before model, tool, memory, or network calls. See [`Policy`](../specs/kubernetes/policy.md).

## R

**RAG** — Retrieval-Augmented Generation. Joch tracks RAG indices, the knowledge sources that feed them, and every retrieval as part of the trace. See [`RAG`](../specs/kubernetes/rag.md).

## S

**Secret** — An external secret reference (Vault, Kubernetes secret, AWS Secrets Manager, env). Joch never stores secret values directly. See [`Secret`](../specs/kubernetes/secret.md).

**SPDX** — A Linux Foundation BOM standard supported by Joch's AgBOM emitter.

**StateCheckpoint** — A vendor-neutral snapshot of agent state used during provider migration. See [`StateCheckpoint`](../specs/kubernetes/state-checkpoint.md).

**SWID** — A NIST software identification standard supported by Joch's AgBOM emitter.

## T

**Team / Namespace** — A multi-tenant boundary in Joch. See [`Team / Namespace`](../specs/kubernetes/team-namespace.md).

**Tool** — A single callable function exposed through the tool gateway. See [`Tool`](../specs/kubernetes/tool.md).

**ToolCall** — One concrete invocation of a tool, with side-effect classification, idempotency key, approval status, inputs, and outputs. See [`ToolCall`](../specs/kubernetes/toolcall.md).

**Trace** — The structured event log of an execution. See [`Trace`](../specs/kubernetes/trace.md) and [Events](../aos/events.md).

**Trace (pillar)** — One of the three AOS pillars: every agent decision is captured as an event in OpenTelemetry / OCSF format.
