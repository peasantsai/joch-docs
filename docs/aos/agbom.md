# AgBOM (Inspect)

AgBOM — Agent Bill of Materials — is the **inspect** pillar of OWASP AOS. It provides a structured, dynamic inventory of every component comprising an agent system: tools, models, capabilities, knowledge sources, memory, and dependencies.

> AgBOM dynamically adapts to reflect the rapid iteration and evolution of agent architectures, especially in real-time or distributed environments. — [OWASP AOS](https://aos.owasp.org/)

Joch implements AgBOM through the [`ABOM`](../specs/kubernetes/abom.md) resource. Each agent has its own ABOM record; the record is regenerated on every change and emitted in CycloneDX, SPDX, and SWID.

## Why AgBOM exists

AgBOM enables developers, auditors, and stakeholders to determine:

- which tools, models, and capabilities are embedded within an agent,
- who authored each component,
- what version and configuration is currently deployed,
- what external services and data sources are accessed.

This visibility supports security tracing, version tracking, and regulatory compliance. It is the agent equivalent of an SBOM for application software.

## Joch's tracked entities

Per OWASP AgBOM, the following entity classes are tracked. Joch additionally tracks framework adapter and policy versions because they are part of an agent's effective configuration.

| Entity | Parameters captured |
|---|---|
| Standard Packages | Name, Description, Version |
| Models | Name, Version, Description, Endpoint, Context Window, Args |
| Capabilities | Agent Card (per A2A), discovered Agents, MCP servers (`protocolVersion`, `capabilities`, `serverInfo`) |
| Knowledge | Name, Description, Schema, Search type, Search args |
| Memory | Name, Description, Type, Size, Search args, Window size, Path |
| Tools | Name, Description, Scheme, Endpoint (local / directly attached / MCP) |
| Framework adapter | Name, Version, Capability vector |
| Policies | Name, Version, Selectors |

## Refresh triggers

AgBOM is dynamic. Joch refreshes it when any of the following change:

- the agent capability set,
- an MCP server's discovered tools, resources, or version,
- a knowledge source or RAG index,
- a tool record or its endpoint,
- a memory record,
- a model record or model route,
- a policy that applies to the agent,
- the framework adapter version.

The `ABOM.refresh.onChange` flag enables automatic regeneration. `ABOM.refresh.schedule` adds a periodic refresh as a safety net.

## Output formats

| BOM standard | Joch support |
|---|---|
| [CycloneDX](https://cyclonedx.org/) | full — see [mapping](cyclonedx-mapping.md) |
| [SPDX](https://spdx.dev/) | full |
| [SWID](https://csrc.nist.gov/Projects/Software-Identification-SWID) | full |

## Operator commands

```bash
joch abom support-triage
joch abom support-triage --format cyclonedx > support-triage.cdx.json
joch abom support-triage --format spdx     > support-triage.spdx.json
joch abom support-triage --format swid     > support-triage.swid.xml
joch abom support-triage --diff --from 16 --to 17
joch abom ls --high-risk
```

The `--high-risk` filter surfaces agents whose ABOM contains components below trust thresholds, with unpinned MCP servers, or with policy violations.

## Signing

Joch ABOM emissions can be signed with a configured key:

```yaml
signing:
  enabled: true
  keyRef:
    name: joch-abom-signing-key
```

Signed BOMs are written alongside the unsigned ones; downstream consumers can verify provenance.

## Audit and compliance

For regulated industries, Joch ABOM forms the basis of agent supply-chain audits. Auditors can request:

- the ABOM at the time of any historical execution (via `Execution.status.abomSnapshotRef`),
- the diff between two ABOM generations,
- the full set of tools, MCP servers, and knowledge sources reachable from a given agent at a given date.

This satisfies the **inspect** pillar of AOS for audit-class compliance.
