# Competitive Landscape

Joch's category is **agent operations control plane**. The most common false comparisons place Joch against vendor SDKs or generic observability tools. The taxonomy below clarifies where Joch fits.

## Adjacent categories

| Category | Examples | Relationship to Joch |
|---|---|---|
| Agent SDKs | OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework | Joch sits **above**: governs and operates agents the SDKs build. |
| Workflow / multi-agent frameworks | LangGraph, CrewAI | Same: Joch governs them. |
| Generic LLM observability | Langfuse, Helicone, Phoenix, Arize, Honeycomb GenAI | Joch ships traces here as the OTel target; it does not compete on visualization. |
| MCP gateways / tool registries | early projects, vendor offerings | Joch's [MCP Gateway](../architecture/mcp-gateway.md) is a strict superset (registry + firewall + scanning + AOS). |
| Model gateways / routers | LiteLLM, Portkey, OpenRouter | Joch's [Model Router](../architecture/model-router.md) is integrated, but Joch can also route through these as backends. |
| Policy / authorization | OPA, Cedar, Kyverno | Joch's [Policy Engine](../architecture/policy-engine.md) compiles to these as targets; the source of truth is the portable Joch resource. |
| AOS Guardian Agents | new category | Joch's policy engine is the default Guardian; third-party Guardians compose. |
| AI BOM / supply-chain | nascent | Joch is the canonical AOS AgBOM emitter for agent fleets. |
| Platform engineering | Backstage, Port | Backstage catalogs services; Joch catalogs and operates **agents**. Complementary. |
| FinOps / cost analytics | Vantage, CloudZero, AWS Cost Explorer | Joch exports cost; FinOps tools consume it for org-wide dashboards. |

## The capability matrix (recap)

The full capability matrix lives in [Comparison](../concepts/comparison.md). Summarized:

- vendor SDKs do **build**, not **operate**;
- generic LLM observability tools do **traces**, not **inventory + policy + ABOM**;
- model gateways do **routing**, not **governance + ABOM + state**;
- MCP-only gateways do **MCP**, not **the cross-SDK control plane**;
- platform engineering tools catalog **services**, not **agents and their dependencies**.

## Coexistence patterns

Joch is built to coexist with the tools customers already run. Common coexistence patterns:

- **Joch + Honeycomb / Datadog / Grafana** — Joch exports OTLP into the customer's existing observability backend.
- **Joch + Splunk / Elastic / SIEM** — Joch exports OCSF for security-relevant events.
- **Joch + LiteLLM / Portkey** — these can sit behind the Joch Model Router as provider abstractions; Joch retains policy and audit.
- **Joch + OPA / Cedar** — Joch policies compile to Rego or Cedar where customers already run those engines.
- **Joch + Backstage / Port** — Joch surfaces agent metadata via catalog plugins.

Coexistence is not a posture; it is a design rule. Joch is the operations control plane, not a one-stop replacement.

## Where vendors might enter

Each major SDK vendor (OpenAI, Anthropic, Google, Microsoft) could add fleet-management surfaces. Two facts soften that risk:

1. Vendor fleet tools cover the vendor's SDK by definition. The fleet-management problem is **cross-vendor** by design — the customer needs the same view across all SDKs they use, not separate views per vendor.
2. AOS conformance is a cross-vendor coordination point. Once AOS becomes the standard, single-vendor "control planes" become harder to defend than they look.

## The honest summary

Joch wins where customers run **more than one SDK**, **more than one vendor**, or **agents that must satisfy compliance and audit beyond a single vendor's controls**. That is rapidly becoming the median enterprise profile, not the edge case.
