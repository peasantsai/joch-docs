# Revenue Models

Joch's revenue model is **open-core SaaS with optional self-hosted enterprise**. The Apache-2.0 core is generous and complete enough to run production fleets. Commercial offerings add multi-tenant operations, premium connectors, support, and compliance packaging.

## Tiers

### Joch Open Source

```text
License:        Apache-2.0
Cost:           free
Distribution:   GitHub releases, container images, Helm chart, Homebrew
Includes:       full control plane, gateways, AOS exporters, CLI, console,
                framework adapters, provider adapters, Kubernetes operator
Support:        community (GitHub issues, Discussions)
SLA:            none
```

Sufficient for self-hosted production deployments. Independent forks are allowed by license.

### Joch Cloud — Starter

```text
Cost:           free for small teams (limit by agents, executions, retention)
Includes:       hosted multi-tenant control plane, hosted Console,
                managed traces and AgBOM storage, customer-runtime tunnel,
                community-vetted MCP / tool catalog
Support:        community + best-effort response
SLA:            none
```

The free tier exists to remove the hosting friction for teams just starting.

### Joch Cloud — Team

```text
Cost:           per-agent monthly + per-execution overage tiers
Includes:       Starter + SSO/SCIM, RBAC, premium connectors,
                multi-region trace and AgBOM retention up to 90 days,
                Slack / email approval routing, audit export to SIEM
Support:        business-hours email + chat
SLA:            99.9% control plane
```

This is the typical landing tier for engineering organizations.

### Joch Cloud — Enterprise

```text
Cost:           contract; capacity-based or org-wide
Includes:       Team + advanced policy packs, signed MCP / tool marketplace,
                dedicated control plane in the customer cloud account,
                FedRAMP / SOC 2 / ISO 27001 attestations, advanced eval
                packs, custom integrations
Support:        24x7 with named TAM
SLA:            99.95% control plane; 1h Sev-1 response
```

For regulated industries and large fleets where airgap or dedicated hosting matters.

### Joch Enterprise (self-hosted)

```text
Cost:           annual subscription tied to control plane capacity
Includes:       all Joch Cloud features in a self-hosted distribution,
                airgap installer, hardened images, FedRAMP packaging,
                dedicated patch / CVE channel
Support:        24x7 with named TAM
SLA:            99.95% within the customer's runtime
```

For customers that cannot use SaaS at all.

## Add-ons

```text
Premium MCP / Tool Marketplace        per server / month
Compliance pack (SOC 2 / HIPAA / FedRAMP exports)
Custom Guardian Agent integration
Joch Registry mirror with private repositories
```

## Pricing principles

- **Pay for control-plane usage, not for inference.** Inference billing stays with the provider; Joch is a multiplier on operations efficiency, not on token cost.
- **Charge for agent count and execution scale**, not per seat. The seat model under-counts platforms with many automations and few humans.
- **No per-feature paywalls in OSS** for things that should be obviously free — basic AgBOM, traces, hooks, gateways, and policies are open.
- **Trace / AgBOM retention is a real cost** and is metered explicitly in tiers.

## Why open-core works here

- The OSS core is complete enough for adoption to compound: customers deploy Joch alongside their SDKs without procurement.
- The commercial value lives in **scale, multi-tenancy, integrations, and compliance** — not in feature crippling.
- AOS standards alignment means Joch interoperates with third-party Guardians, BOM consumers, and SIEMs, so vendor lock-in is not a buying objection.

The model is similar in shape to GitLab, HashiCorp's pre-BSL era, and Backstage with Spotify Portal. It is well-understood by buyers and well-supported by the developer community.

## Risk and mitigation

| Risk | Mitigation |
|---|---|
| Hyperscaler bundles control-plane equivalent into platform offers | Stay multi-cloud and multi-vendor by design; AOS conformance prevents single-vendor lock-in. |
| Open-source forks fragment the ecosystem | Generous OSS, governed canonical repo, trademark on the name; forks may exist, the network advantage stays with the canonical project. |
| Compliance certifications take time | Sequenced: SOC 2 Type 1 → SOC 2 Type 2 → ISO 27001 → FedRAMP. Each unlocks a tier. |
| Customers demand on-prem | Joch Enterprise self-hosted is a first-class distribution, not an afterthought. |

## Honest summary

The best business outcome is to be the **default** AI agent control plane. Open source is the way there. Pricing is honest, metered, and tied to operational value, not artificial scarcity.
