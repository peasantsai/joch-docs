# Brand

## Name

**Joch** — pronounced */jok/*. A Joch (or yoke) is the wooden frame used to bind and guide draft animals so they pull together. The product metaphor is direct: Joch harnesses AI agents into coordinated working fleets.

## Tagline

> **Joch — the vendor-neutral control plane for AI agent fleets.**

## Brand architecture

```text
Company / org      PeasantsAI
Product            Joch
CLI                joch
Cloud              Joch Cloud
Enterprise         Joch Enterprise
Registry           Joch Registry
Marketplace        Joch Marketplace
Console            Joch Console
Operator           Joch Operator
```

Domain naming, in priority order:

```text
joch.dev      primary
joch.run      reserved
joch.ai       reserved
peasantsai.io company / org
```

## API group

```yaml
apiVersion: joch.dev/v1alpha1
kind: Agent
```

The `joch.dev` API group is short, recognizable, and aligned with the brand.

## CLI surface

```bash
joch up
joch apply -f agent.yaml
joch get agents
joch describe agent support-triage
joch run support-triage "Triage incoming queue"
joch trace exec-20260510-001
joch approvals ls
joch mcp ls
joch agbom support-triage
joch promote support-triage --from staging --to prod
joch cost by-team --since 7d
```

A Kubernetes-style alias `jochctl` is reserved for users who prefer that convention; the canonical command is `joch`.

## Voice

- Operator-grade: write for the platform engineer, the security engineer, the ML engineer in the trenches.
- Specific over abstract: prefer "tool calls flow through the gateway" over "agents are governed."
- Honest about scope: state what Joch is and is not. The [positioning](../concepts/positioning.md) page is the canonical reference.

## What we are not

- We are not a model provider.
- We are not an agent SDK.
- We are not a notebook or playground.
- We do not fork or replace the SDKs our customers use.

When in doubt, the operator-language test applies: would a senior platform engineer recognize this sentence as something they would write in a runbook? If not, simplify.

## Trademark

**Joch**, **Joch Cloud**, **Joch Enterprise**, **Joch Console**, **Joch Operator**, **Joch Registry**, and **Joch Marketplace** are trademarks of PeasantsAI. Forks of the open-source project are allowed under Apache-2.0; they may not use these names in branding or marketing.
