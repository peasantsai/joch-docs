# Go-To-Market

The go-to-market motion is **bottom-up developer adoption**, **content and standards leadership**, and **enterprise pull through compliance**, in that order.

## The wedge

> *"Joch is the secure MCP/tool-call gateway and inventory layer for all your AI agents — regardless of whether they are built with OpenAI, Claude, Gemini, Microsoft, LangGraph, or custom code."*

This is sharper than "agent fleet manager" and lands a real day-one outcome:

- a security team or platform team can install Joch in a day,
- register every MCP server,
- pin and scan them,
- and immediately reduce supply-chain risk for the rest of the org.

From the wedge, Joch expands into inventory, policy, state, evals, and release gates as the customer's needs grow.

## The motion

### Phase 1 — Awareness via standards (months 0-6)

- AOS conformance from day one. Public mapping pages: [AgBOM](../aos/agbom.md), [Hooks](../aos/hooks.md), [Events](../aos/events.md).
- Engineering blog posts and conference talks on cross-SDK governance, MCP supply chain, and ABOM.
- Open registry and example agents in `peasantsai/joch-examples` and `peasantsai/joch-registry`.
- Visible position in the OWASP AOS, MCP, and OpenTelemetry GenAI working groups.

### Phase 2 — Bottom-up adoption (months 3-12)

- One-line install: `brew install joch && joch up`.
- Compose stack and Helm chart for self-hosted teams.
- Adapter parity for the four major SDKs (OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework) plus LangGraph and CrewAI.
- Frictionless free tier on Joch Cloud (Starter).

### Phase 3 — Enterprise pull (months 9+)

- SOC 2 Type 1 → Type 2 progression.
- ISO 27001 readiness.
- Premium MCP / tool marketplace with signing.
- Slack / email / Microsoft Teams approval routing.
- Dedicated TAM and named-account motion for regulated industries.

### Phase 4 — Public sector and FedRAMP (months 18+)

- Airgap distribution.
- FedRAMP Moderate readiness.
- Joch Enterprise self-hosted offering.

## Channels

| Channel | Role |
|---|---|
| GitHub | Primary distribution; the truth of the project. |
| Documentation site (this) | Buyer-grade and operator-grade reference. |
| Standards bodies | OWASP AOS, MCP, OpenTelemetry GenAI — visible contribution. |
| Conferences | KubeCon, RSAC, OWASP Global, AI Engineer Summit, GitHub Universe, security conferences. |
| Slack and Discord communities | Direct support and roadmap signal. |
| Hyperscaler marketplaces | AWS, Azure, GCP marketplaces for Cloud and Enterprise. |
| Cloud partner ecosystem | OEM / co-sell relationships with platform vendors. |

## Partnerships

- **SDK vendors** — Joch is complementary, not competitive; we ship adapters early and work with each vendor's developer relations team.
- **MCP server publishers** — vetted servers list on the Joch marketplace with signing.
- **Observability vendors** — Joch exports OTLP and OCSF; we ship integrations with Honeycomb, Datadog, Grafana, Splunk, Elastic.
- **Identity providers** — Okta, Microsoft Entra, Google Workspace, Auth0 SSO/SCIM integrations.
- **System integrators and consultancies** — implementation partners for regulated industries.

## Sales motion

- **Self-serve** for teams up to a defined cap (free or Team).
- **Inside sales** for mid-market and growing teams.
- **Field sales** for enterprise and regulated industries.

The motion is intentionally not "demo-and-quote." Joch must be installable and useful in an hour without sales contact. Field sales handles the additional weight of enterprise procurement, not the discovery.

## Metrics

| Stage | KPI |
|---|---|
| Awareness | GitHub stars, docs visits, standards-track issues touched |
| Adoption | unique install fingerprints, agents registered per install, framework adapters used |
| Activation | weekly active operators, traces ingested, ABOMs generated |
| Conversion | Joch Cloud sign-ups, Team conversions, Enterprise pipeline |
| Retention | net revenue retention, agent count growth, log volume growth |

## Honest summary

Joch wins by being **easier to adopt than any single-vendor control plane**, **harder to remove once adopted**, and **standard-compliant enough that procurement does not block it**. Bottom-up first, top-down later, and avoid the trap of marketing as a feature checklist.
