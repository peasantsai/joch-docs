# AOS Conformance

Joch implements the **[OWASP Agent Observability Standard (AOS)](https://aos.owasp.org/)** as its conformance baseline. AOS makes AI agents trustworthy by standardizing three properties: agents must be **inspectable**, **traceable**, and **instrumentable**.

> AOS makes agents an open book: they have a dynamic bill of materials, a clear audit trail, and hard inline controls. — [OWASP AOS](https://aos.owasp.org/)

Joch's data plane is a faithful AOS implementation. Joch agents emit AgBOMs, expose AOS hooks, and emit AOS-aligned trace events. Third-party Guardian Agents, BOM consumers, and trace backends interoperate without bespoke integration.

## The three pillars

<div class="grid cards" markdown>

-   :material-magnify-scan: **Inspect**

    ---

    Agents publish a current Agent Bill of Materials (AgBOM) extending CycloneDX, SPDX, and SWID. Joch's [`AgBOM`](../specs/kubernetes/agbom.md) resource implements this pillar.

    [Read the AgBOM page](agbom.md)

-   :material-hook: **Instrument**

    ---

    Agents expose hooks that a Guardian Agent can use to `allow`, `deny`, or `modify` decisions. Joch's gateways implement the AOS hook contract.

    [Read the hooks reference](hooks.md)

-   :material-graph-outline: **Trace**

    ---

    Agents emit comprehensive events on every runtime decision and lifecycle change. Joch's [`Trace`](../specs/kubernetes/trace.md) resource implements this pillar.

    [Read the events reference](events.md)

</div>

## Industry-standard mappings

Per AOS, Joch does not invent new BOM or telemetry standards — it extends industry-standard ones:

| Standard | Joch surface | Status |
|---|---|---|
| [CycloneDX](https://cyclonedx.org/) | AgBOM emission | full support |
| [SPDX](https://spdx.dev/) | AgBOM emission | full support |
| [SWID](https://csrc.nist.gov/Projects/Software-Identification-SWID) | AgBOM emission | full support |
| [OpenTelemetry](https://opentelemetry.io/) | Trace export | full support |
| [OCSF](https://ocsf.io/) | Security event export | full support |

Mappings:

- [CycloneDX Mapping](cyclonedx-mapping.md)
- [OpenTelemetry Mapping](opentelemetry-mapping.md)
- [OCSF Mapping](ocsf-mapping.md)

## Roles

AOS distinguishes two agent roles:

- **Observed Agent** — the agent being instrumented. In Joch, every agent built on a [framework adapter](../architecture/framework-adapters.md) is an Observed Agent.
- **Guardian Agent** — the policy enforcement entity. Joch's [policy engine](../architecture/policy-engine.md) plays the Guardian role by default; third-party Guardian Agents can be plugged in.

The Observed → Guardian relationship is the channel through which `allow`, `deny`, and `modify` decisions flow.

## Where Joch goes beyond AOS

Joch implements AOS faithfully but adds operations features that AOS leaves to implementations:

- a portable [`Policy`](../specs/kubernetes/policy.md) language and engine,
- a cross-SDK agent inventory keyed off framework adapters,
- a tool gateway and MCP gateway with version pinning, scanning, and idempotency,
- a model router with capability- and cost-aware fallback,
- a release management surface (`Eval`, `Approval`, promotion gates).

These build on the AOS contract; they do not replace it.

## Conformance matrix

| AOS requirement | Joch implementation |
|---|---|
| Standard hooks | [Tool Gateway](../architecture/tool-gateway.md), [MCP Gateway](../architecture/mcp-gateway.md), [Model Router](../architecture/model-router.md), [Memory](../architecture/data-plane.md), [RAG](../architecture/data-plane.md) |
| `allow` / `deny` / `modify` decisions | [Policy Engine](../architecture/policy-engine.md) |
| Standardized trace | [`Trace`](../specs/kubernetes/trace.md) + OpenTelemetry / OCSF |
| Comprehensive AgBOM | [`AgBOM`](../specs/kubernetes/agbom.md) + CycloneDX / SPDX / SWID |
| Dynamic AgBOM refresh | [`AgBOM.refresh.onChange`](../specs/kubernetes/agbom.md) and triggers |
| MCP integration | [MCP Gateway](../architecture/mcp-gateway.md) and [`MCPServer`](../specs/kubernetes/mcpserver.md) |
| A2A integration | [`Handoff`](../specs/kubernetes/handoff.md) and the A2A broker |
