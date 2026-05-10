# Open-Core Strategy

Joch is open-core: the control-plane core is Apache-2.0 and complete enough to run production fleets; commercial offerings add multi-tenant SaaS, premium connectors, marketplaces, and compliance packaging.

## Principles

1. **Generous core.** The OSS surface must be sufficient for real production deployments. A crippled OSS is a one-quarter advantage and a multi-year reputation cost.
2. **No paywalled basics.** Inventory, traces, ABOM, hooks, gateways, model routing, and core policy belong in the OSS.
3. **Standards-first.** AOS conformance, OpenTelemetry, OCSF, CycloneDX, SPDX, SWID — all in OSS. Standards alignment is not a moat; it is table stakes.
4. **Honest boundary.** What is commercial is so because it is **operationally** different (multi-tenant, hosted, signed, compliance-attested), not because we removed a feature from OSS.
5. **Single trademark.** "Joch" is the canonical project; forks may exist by license, but they cannot use the trademark.

## What's open source (Apache-2.0)

```text
joch CLI
joch-server (apiserver, store, admission, policy engine, controller mgr, compiler, scheduler)
joch-worker (runtime worker)
joch-gateway (tool gateway, MCP gateway, secret broker)
joch-router (model router)
joch-memory (memory + RAG)
joch-trace (OTLP + OCSF exporters)
joch-abom (CycloneDX + SPDX + SWID emitters)
joch-operator (Kubernetes CRDs + controllers)
joch console (web UI)
framework adapters (one repo per SDK)
provider adapters (one repo per provider)
SDKs (Python, TypeScript)
Helm chart
Compose stack
example agents and registry seeds
```

## What's commercial

Joch Cloud and Joch Enterprise add features that are operationally complex or compliance-sensitive:

```text
Multi-tenant control plane (cross-org, hosted)
Hosted Console with org-wide views
SOC 2 / ISO 27001 / FedRAMP attestations
Cross-tenant policy templates and benchmarking
Premium connectors (SSO, SCIM, identity providers, ITSM)
Vetted and signed MCP / tool marketplace
Advanced policy packs (PII, HIPAA, financial-services)
Airgap installer and hardened images (Joch Enterprise)
Dedicated tenant in customer cloud account (Enterprise)
24x7 support and SLAs
```

These cannot be cleanly given away because they require ongoing security, hosting, and compliance investment.

## Trademark and naming

The project is published under PeasantsAI. The name **Joch** and the logo are trademarks. The license is Apache-2.0 for code; the trademark is reserved per a published [Trademark Policy](#).

Forks of the OSS core are allowed under Apache-2.0. Forks may not use the **Joch** name in branding or marketing. The trademark protects users from confusion and protects the project from dilution.

## Governance

Joch's governance is documented in [Project Governance](../community/governance.md). The summary:

- A neutral steering committee with industry representation.
- Public RFC process for major changes.
- Apache-2.0 contributor license.
- Public roadmap and quarterly updates.

The governance model is designed so that the canonical project is genuinely community-maintainable, even when commercial features ship in parallel.

## Why this works

Two forces sustain the model:

1. **Network effect of the OSS** — registry contributions, framework adapters, AOS exporters, and operator views accrue in the canonical project because contributors find the audience there.
2. **Operational asymmetry of the commercial product** — hosting a multi-tenant SaaS, signing tools, and maintaining FedRAMP packaging is not amenable to volunteer contribution at scale. That is where the company captures revenue.

The model is similar in shape to GitLab, HashiCorp's pre-BSL era, and Backstage with Spotify Portal. It is well-understood by buyers and well-supported by the developer community.

## What we will *not* do

- Relicense to BSL, SSPL, or "source-available." We learned from the cautionary examples and intend to stay Apache-2.0.
- Hold features hostage in OSS to coerce upgrades. Commercial value comes from new operational capability, not from withholding existing OSS features.
- Build closed-source patches into the OSS code path. Commercial features compose **above** the OSS, never inside it.

## Honest summary

The OSS is the product for many users. The commercial product is for operators, enterprises, and regulated industries that want hosted, signed, and attested operations. The boundary is honest, durable, and aligned with how customers actually consume software.
