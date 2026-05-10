# Data Plane

The Joch data plane is the set of services that sit on the boundary between agent runtimes and the world: tools, MCP servers, models, memory, RAG indices, and other agents. Every cross-boundary action passes through a data-plane service so the policy engine can enforce rules and so observability captures the event.

## Services

```text
joch-tool-gateway         enforces ToolCall policies; AOS toolCallRequest / toolCallResult
joch-mcp-gateway          registers, pins, scans, and proxies MCP servers; AOS protocols/MCP
joch-model-router         routes to providers via ModelRoute; AOS message
joch-memory               working / semantic / episodic memory; AOS memoryContextRetrieval / memoryStore
joch-rag                  retrieval over indices; AOS knowledgeRetrieval
joch-artifact             durable artifact storage by reference
joch-trace                ingest, store, and export trace events
joch-runtime-worker       runs framework adapters and agent processes
joch-secret-resolver      resolves secret refs at the call boundary
joch-a2a-broker           records inbound and outbound A2A messages
```

Data-plane services are stateless except for memory, RAG, artifact, and trace, all of which are pluggable storage backends.

## Network topology

In Docker:

```text
joch-runtime-worker → joch-tool-gateway → external APIs
                    → joch-mcp-gateway  → MCP servers
                    → joch-model-router → provider APIs
                    → joch-memory
                    → joch-rag
                    → joch-artifact
                    → joch-trace
                    → joch-a2a-broker
```

In Kubernetes:

```text
ServiceAccount: joch-worker
  may call: tool-gateway, mcp-gateway, model-router, memory, rag, artifact, trace, a2a-broker
  may not call: provider APIs directly, MCP servers directly, Vault, customer DBs
NetworkPolicy: enforces this in-cluster.
```

## Why the gateway boundary exists

The single most valuable property of the data plane is that **agents never talk directly to providers, tools, or MCP servers**. Every crossing is a chance to enforce policy, redact data, audit, and budget.

Concretely, the boundary lets Joch:

- enforce a portable [`Policy`](../specs/kubernetes/policy.md) regardless of SDK,
- present uniform AOS hooks (`allow` / `deny` / `modify`) to a Guardian Agent,
- attribute cost per tool call, model call, and memory operation,
- redact PII before tool arguments leave the gateway,
- pin MCP server versions and detect schema drift,
- block egress to non-allowlisted destinations,
- replay any execution from the event log.

## Hooks at every gateway

Each data-plane gateway implements the OWASP AOS hook contract:

| Gateway | AOS hooks |
|---|---|
| Tool Gateway | `steps/toolCallRequest`, `steps/toolCallResult` |
| MCP Gateway | `protocols/MCP` (outbound and inbound) |
| Model Router | `steps/message` (user and agent), `steps/agentTrigger` |
| Memory | `steps/memoryContextRetrieval`, `steps/memoryStore` |
| RAG | `steps/knowledgeRetrieval` |
| A2A Broker | A2A hooks (`send_message`, `stream_message`, `cancel_request`, `get_task`, notification config get/set, `resubscribe`) |

The hook contract is defined in [Hooks](../aos/hooks.md). The Guardian Agent role — the entity that returns `allow` / `deny` / `modify` — is filled by Joch's [Policy Engine](policy-engine.md) by default and can be replaced by a third-party Guardian Agent.

## Trace fan-out

Every gateway emits trace events. The trace path is:

```text
gateway → joch-trace (ring buffer + durable store)
                   │
                   ├─→ OpenTelemetry collector (OTLP)
                   ├─→ OCSF sink
                   └─→ Joch operator views (top agents, drift, denials, ...)
```

See [Observability](../pillars/observability.md) and [Events](../aos/events.md).

## Artifact handling

Tool outputs, generated reports, datasets, transcripts, and similar payloads are stored by `joch-artifact` and referenced from traces, conversations, and tool calls:

```text
artifact://run/exec-123/report-final.md
artifact://tool-results/call-abc123/raw.json
artifact://model-output/exec-123/turn-7.json
```

Backing stores are pluggable: local filesystem, S3, GCS, MinIO, Azure Blob.

## What the data plane does *not* do

- It does not write desired state. That belongs to the [Control Plane](control-plane.md).
- It does not own the agent loop. That belongs to the SDK runtime via a [framework adapter](framework-adapters.md).
- It does not generate prompts or plans. That belongs to the SDK.

The data plane's job is to make every external interaction safe, attributable, and replayable.
