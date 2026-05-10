# Target Audience

Joch is built for organizations operating **more than one agent**, **across more than one SDK or vendor**, that need **inventory, governance, and audit** beyond what individual SDKs provide.

## ICP — primary

| Profile | Why Joch |
|---|---|
| Mid-size SaaS engineering org | Multiple agents in production across teams; needs cost control, evals, and approvals before customers notice regressions. |
| Enterprise platform team | Owns the developer experience for AI agent teams; needs a Backstage-style catalog with control-plane teeth. |
| Regulated industries (financial services, healthcare, public sector) | Needs ABOM, audit, residency controls, and SIEM integration that vendor SDKs do not provide on their own. |
| AI-native products with agent fleets | Needs cross-vendor model routing, MCP governance, and release gates so growth does not regress quality. |

## Buyer personas

### VP / Director of Platform Engineering

Owns developer productivity and shared infra. Buys Joch to:

- give every team the same agent operations surface, regardless of SDK,
- prove ROI of platform investment via cost / quality observability,
- consolidate vendor and tool sprawl into one governed boundary.

Pain points: agent sprawl, inconsistent guardrails, no shared catalog, "every team reinvents" syndrome.

### Head of AI / Director of ML Engineering

Owns AI delivery and quality. Buys Joch to:

- ship agent updates safely with eval gates,
- migrate across providers when economics or capabilities shift,
- avoid lock-in to a single SDK ecosystem.

Pain points: brittle releases, surprise outages, no portability, "the agent regressed" incidents.

### CISO / Head of Security & Compliance

Owns risk posture. Buys Joch to:

- enforce policy on every external action,
- keep an immutable audit log of agent decisions,
- get ABOM and OCSF events for the SIEM,
- reduce MCP supply-chain risk.

Pain points: no visibility into agent decisions, MCP server proliferation, lack of agent-specific compliance evidence.

### Champion: Senior Engineer / Tech Lead

The day-zero advocate. Wants Joch because:

- they already run more than one SDK and feel the operational pain,
- they want trace-grade visibility without coding it themselves,
- they would rather adopt one open standard (AOS) than five proprietary control surfaces.

Pain points: lack of cross-SDK observability, no policy portability, fragile prompt-and-tool changes.

### End user: Operator / Approver / Auditor

Uses Joch in their daily flow:

- the **operator** runs `joch get agents`, `joch trace`, `joch top tools`,
- the **approver** triages `joch approvals ls` from Slack or the console,
- the **auditor** consumes ABOM, OCSF, and release records in the SIEM and audit workflows.

## Anti-personas (where Joch is the wrong tool)

- **Solo developers building one agent.** A vendor SDK is enough.
- **Hobbyists and student projects.** No control-plane need.
- **Single-vendor shops with no compliance or cost pressure.** The moat is in coverage; absent multi-vendor or compliance pressure, a vendor SDK plus a basic trace exporter suffices.
- **Teams that want a managed agent runtime.** Vendor SaaS (Foundry, ChatGPT Enterprise, Vertex Agents) is a closer fit. Joch is a control plane, not a hosted runtime.

## Sizing the market

A useful proxy: organizations that already operate **N > 1** agents and have at least one of (compliance pressure, cost pressure, multi-SDK or multi-vendor exposure). That cohort is growing at the speed of agent adoption itself, which means the addressable population is expanding monthly rather than annually.

The buying signals are visible in industry reports, conference talks, and hiring pages: when an organization posts a "platform team for AI agents" role, they are an ICP fit.
