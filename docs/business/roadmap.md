# Roadmap

The Joch roadmap is sequenced by **wedge first, breadth second, compliance third**. Dates are aspirational; sequence is committed.

## Now (next quarter)

- **MCP Gateway hardening**: registry, version pinning, schema-drift detection, sandboxing, prompt-injection scanning of inbound tool results.
- **Tool Gateway with AOS hooks**: `toolCallRequest`, `toolCallResult`, side-effect classification, idempotency, approvals.
- **Framework adapters**: OpenAI Agents SDK, Claude Agent SDK, custom-python.
- **Model Router** with `ModelRoute` fallback, OpenAI / Anthropic provider adapters.
- **ABOM v1**: CycloneDX emission, sign-on-demand, change-triggered refresh.
- **Trace v1**: OpenTelemetry exporter with GenAI semantic conventions.
- **CLI**: `joch up`, `apply`, `get`, `describe`, `trace`, `abom`, `mcp`, `approvals`, `cost`.
- **Helm chart and Compose stack** for first-day install.

## Next (1–2 quarters out)

- **Framework adapters**: Google ADK, Microsoft Agent Framework, LangGraph, CrewAI, custom-typescript.
- **Provider adapters**: Google, Microsoft Azure / Foundry, Ollama, vLLM, llama.cpp.
- **Approvals service** with Slack, email, and webhook routing.
- **SPDX and SWID** ABOM emission.
- **OCSF** trace exporter for the security event subset.
- **Eval service** and release gates wired into `joch promote`.
- **Web Console** for inventory, traces, approvals, ABOM, costs.
- **Joch Cloud Starter** (free tier) with hosted control plane and customer-runtime tunnel.

## Later (3+ quarters out)

- **Joch Cloud Team and Enterprise tiers** with SSO/SCIM, RBAC, audit export, premium connectors.
- **Vetted MCP / tool marketplace** with signing.
- **A2A broker** with full hook coverage (`send_message`, `stream_message`, `cancel_request`, `get_task`, notification config get/set, `resubscribe`).
- **Advanced policy packs**: PII, HIPAA, financial-services.
- **OPA / Cedar compilation targets** for policy.
- **Joch Enterprise (self-hosted)** distribution with airgap installer.
- **SOC 2 Type 1 → Type 2** attestation cadence.
- **ISO 27001** readiness.

## Standards alignment milestones

| Milestone | Quarter |
|---|---|
| AOS Inspect (AgBOM) v1 — CycloneDX | Now |
| AOS Instrument (Hooks) v1 — full surface | Now / Next |
| AOS Trace v1 — OpenTelemetry | Now |
| AOS Trace v1 — OCSF | Next |
| AOS Inspect — SPDX, SWID | Next |
| MCP gateway full conformance with [MCP spec](https://modelcontextprotocol.io/) | Now |
| OTel GenAI semantic conventions tracking | Continuous |

## Governance milestones

- Contributor License published (Apache-2.0).
- Trademark Policy published.
- Public RFC process bootstrapped.
- Steering committee announced.
- Quarterly community calls.
- Public release notes.

## What is *not* on the roadmap

To stay sharp:

- **No proprietary agent loop or planner** that competes with vendor SDKs.
- **No hosted inference**.
- **No closed-source forks** of the OSS code path.
- **No SaaS-only features** that should obviously be in OSS (basic ABOM, traces, hooks, gateways).

The roadmap is honest by design: a control plane wins by being narrow, deep, and durable, not by sprawling into every adjacent space.
