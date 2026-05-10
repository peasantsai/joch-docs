# OpenTelemetry Mapping

Joch trace events extend OpenTelemetry semantic conventions. Joch ships an OTLP exporter that emits spans, events, and metrics directly into any OpenTelemetry collector.

## Span hierarchy

```text
Execution                  root span: one per joch Execution
  AgentRun                 the agent loop within the execution
    ModelCall              one per model call
    ToolCall               one per tool call
      MCP call             nested inside ToolCall when tool is mcp-backed
    MemoryOp               one per memory read or write
    KnowledgeRetrieval     one per RAG retrieval
    HookDecision           short-lived; one per hook decision
  Approval                 one per approval (parallel to AgentRun)
  Handoff                  one per A2A handoff
  ABOMUpdate               one per ABOM regeneration
```

The root `Execution` span carries the trace id propagated to every child.

## GenAI semantic conventions

Joch follows the OpenTelemetry GenAI semantic conventions where they apply:

| Attribute | Joch source |
|---|---|
| `gen_ai.system` | model provider (`openai`, `anthropic`, `google`, ...) |
| `gen_ai.request.model` | requested model id |
| `gen_ai.response.model` | model id reported by the provider |
| `gen_ai.usage.input_tokens` | input tokens, when reported |
| `gen_ai.usage.output_tokens` | output tokens, when reported |
| `gen_ai.request.temperature` | request parameter |
| `gen_ai.request.top_p` | request parameter |
| `gen_ai.response.id` | provider response id |

## Joch attributes

Joch attributes use the `joch.*` namespace.

### Common (every span)

| Attribute | Description |
|---|---|
| `joch.execution.id` | Execution name |
| `joch.agent.name` | Agent name |
| `joch.agent.version` | Agent record version |
| `joch.agent.namespace` | Namespace |
| `joch.framework` | Framework adapter (e.g. `openai-agents-sdk`) |
| `joch.framework.version` | Framework adapter version |
| `joch.environment` | `dev`, `staging`, `prod` |
| `joch.tenant.id` | Tenant id |
| `joch.turn.id` | AOS turn id |
| `joch.step.id` | AOS step id |

### ModelCall span

| Attribute | Description |
|---|---|
| `joch.modelroute.name` | ModelRoute used |
| `joch.modelroute.fallback_index` | 0 for primary, 1+ for fallback |
| `joch.modelcall.cost_usd` | Cost in USD |
| `joch.modelcall.latency_ms` | Latency in ms |
| `joch.modelcall.region` | Region of the provider endpoint |

### ToolCall span

| Attribute | Description |
|---|---|
| `joch.tool.name` | Tool name |
| `joch.tool.type` | `function`, `rest`, `mcp`, `os` |
| `joch.tool.side_effect` | side-effect class |
| `joch.toolcall.id` | ToolCall name |
| `joch.toolcall.idempotency_key` | Idempotency key |
| `joch.toolcall.requires_approval` | boolean |

### HookDecision span

| Attribute | Description |
|---|---|
| `joch.hook.method` | e.g. `steps/toolCallRequest` |
| `joch.hook.decision` | `allow`, `deny`, `modify` |
| `joch.policy.id` | Policy id producing the decision |
| `joch.policy.version` | Policy version |
| `joch.guardian.id` | Guardian Agent id (default: `joch-policy-engine`) |

### Approval span

| Attribute | Description |
|---|---|
| `joch.approval.id` | Approval name |
| `joch.approval.decision` | `approve`, `reject`, `edit` |
| `joch.approval.decided_by` | Decider identity |
| `joch.approval.timeout_minutes` | Configured timeout |

## Events

Span events emit additional payloads that are not span-level attributes:

```text
joch.policy.denied
joch.budget.exceeded
joch.provider.switched
joch.abom.updated
joch.mcp.schema_drift
```

## Metrics

The OTLP exporter also emits metrics that complement spans:

```text
joch.executions.count                    counter
joch.executions.duration_ms              histogram
joch.toolcalls.count                     counter
joch.toolcalls.error_rate                gauge
joch.modelcalls.cost_usd                 counter
joch.modelcalls.fallback_rate            gauge
joch.policy.denials.count                counter
joch.approvals.queue_depth               gauge
joch.approvals.time_to_decision_ms       histogram
joch.abom.updates.count                  counter
```

## Exporter configuration

```yaml
apiVersion: joch.dev/v1alpha1
kind: Trace
spec:
  export:
    openTelemetry:
      enabled: true
      endpointSecretRef:
        name: otel-collector
      headers:
        x-tenant-id: support-platform
      compression: gzip
      retry:
        maxAttempts: 5
        initialIntervalMs: 500
```

The exporter uses standard OTLP-HTTP or OTLP-gRPC. Endpoints, headers, compression, and retry are configurable per `Trace` record.

## Compatibility

Any OpenTelemetry collector or backend can ingest Joch traces. Joch attributes are additive — backends that do not understand `joch.*` simply ignore them.
