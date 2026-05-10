# CycloneDX Mapping

Joch emits per-agent AgBOMs as [CycloneDX 1.6](https://cyclonedx.org/) documents. The mapping below shows how Joch resources translate into CycloneDX components, dependencies, and properties.

## Document shape

```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.6",
  "version": <generation>,
  "metadata": {
    "timestamp": "<UTC ISO-8601>",
    "tools": [
      { "name": "joch-agbom", "version": "<joch-version>" }
    ],
    "authors": [
      { "name": "<owner team>", "email": "<owner email>" }
    ]
  },
  "components": [ ... ],
  "dependencies": [ ... ],
  "signatures": [ ... ]
}
```

`metadata.version` corresponds to the [`AgBOM`](../specs/kubernetes/agbom.md) resource generation, so consumers can detect changes without diffing every component.

## Component mapping

| Joch resource | CycloneDX `type` | Notes |
|---|---|---|
| [`Agent`](../specs/kubernetes/agent.md) | `service` | Top-level component; identified by `urn:agent:<name>`. |
| [`FrameworkAdapter`](../specs/kubernetes/framework-adapter.md) | `library` | Sub-component of the agent; carries adapter version. |
| [`Model`](../specs/kubernetes/model.md) | `machine-learning-model` | One per referenced model; `urn:model:<provider>:<name>`. |
| [`ModelRoute`](../specs/kubernetes/model-route.md) | `service` | Treated as a routing service. |
| [`Tool`](../specs/kubernetes/tool.md) | `tool` | One per referenced tool; carries side-effect class. |
| [`MCPServer`](../specs/kubernetes/mcpserver.md) | `service` | One per registered MCP server; pinned version is the CycloneDX `version`. |
| [`Memory`](../specs/kubernetes/memory.md) | `data` | Includes type, backend, size. |
| [`RAG`](../specs/kubernetes/rag.md) | `data` | Includes vector store and document count. |
| [`KnowledgeSource`](../specs/kubernetes/knowledge-source.md) | `data` | One per source; classification surfaces as a property. |
| [`Policy`](../specs/kubernetes/policy.md) | `library` | Versioned policy in effect at the time of generation. |
| Standard packages (Python / npm / Go / OS) | `library` | Discovered from runtime image. |

## Property conventions

CycloneDX `components[].properties` carries Joch-specific metadata. Convention: `joch.<area>.<key>`.

| Property | Example value |
|---|---|
| `joch.framework` | `openai-agents-sdk` |
| `joch.frameworkVersion` | `0.7.2` |
| `joch.modelRoute` | `research-default` |
| `joch.tenant` | `support-platform` |
| `joch.environment` | `prod` |
| `joch.policyId` | `external-send-requires-approval` |
| `joch.policyVersion` | `v3` |
| `joch.trustScore` | `0.92` |
| `joch.transport` | `streamable_http` |
| `joch.sideEffect` | `external_write` |
| `joch.requiresApproval` | `true` |
| `joch.dataResidency` | `EU` |
| `joch.classification` | `customer-tier` |

## Dependencies

The `dependencies` array models the agent → component graph:

```json
"dependencies": [
  {
    "ref": "urn:agent:support-triage",
    "dependsOn": [
      "urn:framework-adapter:openai-agents-sdk@0.7.2",
      "urn:model:openai:gpt-5-thinking",
      "urn:model:anthropic:claude-sonnet",
      "urn:mcp:github@1.2.0",
      "urn:tool:zendesk.create_ticket",
      "urn:tool:slack.send",
      "urn:rag:support-docs-rag",
      "urn:memory:support-triage-working",
      "urn:policy:external-send-requires-approval@v3"
    ]
  }
]
```

## Full example

```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.6",
  "version": 17,
  "metadata": {
    "timestamp": "2026-05-10T10:00:00Z",
    "tools": [{ "name": "joch-agbom", "version": "1.0.0" }],
    "authors": [{ "name": "support-platform", "email": "support-platform@example.com" }]
  },
  "components": [
    {
      "type": "service",
      "name": "support-triage",
      "version": "1.4.0",
      "bom-ref": "urn:agent:support-triage",
      "properties": [
        { "name": "joch.framework",        "value": "openai-agents-sdk" },
        { "name": "joch.frameworkVersion", "value": "0.7.2" },
        { "name": "joch.modelRoute",       "value": "research-default" },
        { "name": "joch.environment",      "value": "prod" }
      ]
    },
    {
      "type": "machine-learning-model",
      "name": "openai:gpt-5-thinking",
      "version": "5.5",
      "bom-ref": "urn:model:openai:gpt-5-thinking",
      "properties": [
        { "name": "contextWindowTokens", "value": "400000" },
        { "name": "toolCalling",         "value": "true" }
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
        { "name": "joch.sideEffect",       "value": "external_write" },
        { "name": "joch.requiresApproval", "value": "true" }
      ]
    },
    {
      "type": "library",
      "name": "external-send-requires-approval",
      "version": "v3",
      "bom-ref": "urn:policy:external-send-requires-approval@v3",
      "properties": [
        { "name": "joch.policyKind", "value": "Policy" }
      ]
    }
  ],
  "dependencies": [
    {
      "ref": "urn:agent:support-triage",
      "dependsOn": [
        "urn:model:openai:gpt-5-thinking",
        "urn:mcp:github@1.2.0",
        "urn:tool:zendesk.create_ticket",
        "urn:policy:external-send-requires-approval@v3"
      ]
    }
  ],
  "signatures": [
    { "value": "<base64>", "keyId": "joch-agbom-signing-key" }
  ]
}
```

## Compatibility

CycloneDX consumers (Dependency-Track, GitHub Dependency Graph, security scanners) ingest the document directly. Custom Joch properties live under the standard `properties` extension point and are safe to ignore by tools that do not understand them.
