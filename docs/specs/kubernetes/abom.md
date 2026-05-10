# ABOM

The `ABOM` resource is Joch's per-agent **Agent Bill of Materials**. It implements the OWASP AOS [AgBOM](https://aos.owasp.org/) specification — a dynamic, machine-readable inventory of every component an agent depends on — and emits BOMs in CycloneDX, SPDX, and SWID formats.

[Back to the catalog](index.md)

## Why ABOM

ABOM is the **inspect** pillar of OWASP AOS, applied to agents. It answers, for any agent at any time:

- What tools, models, memory, RAG sources, and MCP servers does the agent depend on?
- Who authored each component?
- What version and configuration are currently in use?
- What external services and data sources are reachable through this agent?

This visibility is the foundation of supply-chain integrity, security tracing, version tracking, and regulatory compliance for agent systems.

## Spec

```yaml
apiVersion: joch.dev/v1alpha1
kind: ABOM
metadata:
  name: support-triage-abom
  namespace: support-platform
  labels:
    agent: support-triage
spec:
  agentRef: { name: support-triage }

  formats:
    - cyclonedx
    - spdx
    - swid

  refresh:
    onChange: true
    schedule: "0 * * * *"

  signing:
    enabled: true
    keyRef:
      name: joch-abom-signing-key

  components:
    standardPackages: true
    models: true
    capabilities: true
    knowledge: true
    memory: true
    tools: true
    mcpServers: true
    policies: true
    secrets: false

  exports:
    cyclonedx:
      destinationRef: { name: artifact-store }
      filenameTemplate: "abom/{{ agent }}/{{ generation }}.cdx.json"
    spdx:
      destinationRef: { name: artifact-store }
      filenameTemplate: "abom/{{ agent }}/{{ generation }}.spdx.json"
    swid:
      destinationRef: { name: artifact-store }
      filenameTemplate: "abom/{{ agent }}/{{ generation }}.swid.xml"

status:
  phase: Ready
  generation: 17
  lastGeneratedAt: "2026-05-10T10:00:00Z"
  artifacts:
    cyclonedxRef: artifact://abom/support-triage/17.cdx.json
    spdxRef: artifact://abom/support-triage/17.spdx.json
    swidRef: artifact://abom/support-triage/17.swid.xml
  highRiskComponents:
    - kind: MCPServer
      name: legacy-internal
      reason: trustScore-below-threshold
```

## What components ABOM tracks

Per OWASP AgBOM, the following entity classes are tracked:

| Entity | Parameters captured |
|---|---|
| Standard Packages | Name, Description, Version |
| Models | Name, Version, Description, Endpoint, Context Window, Args |
| Capabilities | Agent Card (per A2A), discovered Agents, MCP servers (`protocolVersion`, `capabilities`, `serverInfo`) |
| Knowledge | Name, Description, Schema, Search type, Search args |
| Memory | Name, Description, Type, Size, Search args, Window size, Path |
| Tools | Name, Description, Scheme, Endpoint (local / directly attached / MCP) |

Joch additionally tracks `Policy` and `FrameworkAdapter` versions because they are part of an agent's effective configuration.

## Refresh triggers

Per OWASP AgBOM, the ABOM is dynamic. Joch refreshes it when any of the following change:

- the agent capability set,
- an MCP server's discovered tools, resources, or version,
- a knowledge source or RAG index,
- a tool record or its endpoint,
- a memory record,
- a model record or model route,
- a policy that applies to the agent,
- the framework adapter version.

The `refresh.onChange` flag enables automatic regeneration. `refresh.schedule` adds a periodic refresh as a safety net.

## CycloneDX example

```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.6",
  "version": 17,
  "metadata": {
    "timestamp": "2026-05-10T10:00:00Z",
    "tools": [
      { "name": "joch-abom", "version": "1.0.0" }
    ],
    "authors": [
      { "name": "support-platform", "email": "support-platform@example.com" }
    ]
  },
  "components": [
    {
      "type": "service",
      "name": "support-triage",
      "version": "1.4.0",
      "bom-ref": "urn:agent:support-triage",
      "properties": [
        { "name": "joch.framework",       "value": "openai-agents-sdk" },
        { "name": "joch.frameworkVersion","value": "0.7.2" },
        { "name": "joch.modelRoute",      "value": "research-default" },
        { "name": "joch.tenant",          "value": "support-platform" }
      ]
    },
    {
      "type": "machine-learning-model",
      "name": "openai:gpt-5-thinking",
      "version": "5.5",
      "bom-ref": "urn:model:openai:gpt-5-thinking",
      "properties": [
        { "name": "contextWindowTokens", "value": "400000" },
        { "name": "toolCalling",          "value": "true" }
      ]
    },
    {
      "type": "service",
      "name": "github-mcp",
      "version": "1.2.0",
      "bom-ref": "urn:mcp:github@1.2.0",
      "properties": [
        { "name": "joch.trustScore", "value": "0.92" },
        { "name": "joch.transport",  "value": "streamable_http" }
      ]
    },
    {
      "type": "tool",
      "name": "zendesk.create_ticket",
      "version": "v2",
      "bom-ref": "urn:tool:zendesk.create_ticket",
      "properties": [
        { "name": "sideEffect", "value": "external_write" },
        { "name": "requiresApproval", "value": "true" }
      ]
    }
  ],
  "dependencies": [
    {
      "ref": "urn:agent:support-triage",
      "dependsOn": [
        "urn:model:openai:gpt-5-thinking",
        "urn:mcp:github@1.2.0",
        "urn:tool:zendesk.create_ticket"
      ]
    }
  ],
  "signatures": [
    { "value": "<base64>", "keyId": "joch-abom-signing-key" }
  ]
}
```

## Operator commands

```bash
joch abom support-triage
joch abom support-triage --format cyclonedx > support-triage.cdx.json
joch abom support-triage --format spdx > support-triage.spdx.json
joch abom support-triage --format swid > support-triage.swid.xml
joch abom support-triage --diff --from 16 --to 17
joch abom ls --high-risk
```

The `--high-risk` filter surfaces agents whose ABOM contains components below trust thresholds, with unpinned MCP servers, or with policy violations.

## Standards alignment

| Standard | Status |
|---|---|
| [OWASP AOS AgBOM](https://aos.owasp.org/) | implemented |
| [CycloneDX](https://cyclonedx.org/) | full support |
| [SPDX](https://spdx.dev/) | full support |
| [SWID](https://csrc.nist.gov/Projects/Software-Identification-SWID) | full support |

The detailed mapping for each standard lives in the [AOS Conformance](../../aos/index.md) section: [CycloneDX](../../aos/cyclonedx-mapping.md), [OpenTelemetry](../../aos/opentelemetry-mapping.md), [OCSF](../../aos/ocsf-mapping.md).

[Back to the catalog](index.md)
