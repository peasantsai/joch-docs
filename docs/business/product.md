# Product

This page describes the Joch product surface: what is shipped, what is given away, and what is sold.

## Product surfaces

| Surface | Description |
|---|---|
| **Joch Open Source** | Apache-2.0; the control plane core, gateways, framework adapters, AOS-conformant AgBOM and trace exporters, CLI, and Helm chart. Self-hostable end to end. |
| **Joch Console** | Open source web console for inventory, traces, approvals, AgBOM, and cost views. |
| **Joch Registry** | Open registry of reusable agent templates, policies, and tool / MCP definitions. |
| **Joch Cloud** | Commercial multi-tenant SaaS control plane; customer-runtime model; premium connectors; SSO / RBAC; SLA. |
| **Joch Enterprise** | Self-hosted enterprise distribution; advanced policy packs; FedRAMP-ready hardening; airgap support. |
| **Joch MCP / Tool Marketplace** | Vetted MCP servers and tools with SLAs, signing, and security review. |

## What's open source

The Apache-2.0 surface is generous by design: it must be sufficient to run real production fleets, not a crippled trial.

```text
joch CLI                       full
joch-server                    full
joch-worker                    full
joch-gateway (tool + MCP)      full
joch-router (model)            full
joch-memory + joch-rag         full
joch-trace (OTLP + OCSF)       full
joch-agbom (CycloneDX/SPDX/SWID) full
joch-operator (Kubernetes)     full
framework adapters             full (open source per framework)
provider adapters              full (open source per provider)
Joch Console                   full (open source)
```

## What's commercial

Joch Cloud and Joch Enterprise add the operator-grade and multi-tenant features that are not in scope for the OSS core.

```text
Multi-tenant control plane (cross-org)
Hosted SaaS with SOC 2 / ISO 27001 controls
Cross-tenant policy templates and benchmarking
Premium connectors and identity providers
Vetted MCP / tool marketplace + signing service
SSO / SCIM / advanced RBAC
Airgap and FedRAMP packaging
SLA, support, and incident response
```

## What's *not* in the product

- Joch is not an agent SDK. We do not compete with OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, or CrewAI.
- Joch is not a model provider. We do not host inference; we route through providers.
- Joch is not a notebook or playground. We assume teams already author agents in their preferred SDK.

## Standards posture

Joch implements [OWASP AOS](../aos/index.md) faithfully:

- [AgBOM](../aos/agbom.md) → CycloneDX, SPDX, SWID.
- [Hooks](../aos/hooks.md) → standard `allow` / `deny` / `modify` decisions across every gateway.
- [Trace events](../aos/events.md) → OpenTelemetry and OCSF.

This makes Joch interoperable with third-party Guardian Agents, BOM consumers, and SIEMs out of the box.

## Buyer-facing summary

> *"Joch is the vendor-neutral control plane for your AI agent fleet. It runs on the SDKs your teams already use, governs every tool and MCP call uniformly, gives you a versioned AgBOM for every agent, and exports clean OpenTelemetry and OCSF telemetry to whatever you already operate."*
