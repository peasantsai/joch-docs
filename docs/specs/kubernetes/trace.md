# Trace

A `Trace` is the structured event log of one [`Execution`](execution.md). Joch trace events extend OpenTelemetry semantic conventions and OCSF, and they implement the [OWASP AOS Trace specification](../../aos/events.md).

[Back to the catalog](index.md)

## Spec and status

```yaml
apiVersion: joch.dev/v1alpha1
kind: Trace
metadata:
  name: trace-exec-20260510-001
  namespace: support-platform
spec:
  executionRef: { name: exec-20260510-001 }

  sampling:
    mode: full

  export:
    openTelemetry:
      enabled: true
      endpointSecretRef:
        name: otel-collector
    ocsf:
      enabled: true
      eventClasses:
        - 8001  # Application Activity
        - 1003  # Process Activity
      sinkSecretRef:
        name: ocsf-sink

  retention:
    days: 30
    pii:
      redact: true

status:
  phase: Complete
  spans:
    total: 84
    llmCalls: 9
    toolCalls: 18
    handoffs: 2
    hookDecisions: 24
    knowledgeRetrievals: 7
  costUsd: 0.62
  durationMs: 4820
```

## Event taxonomy

Every Joch execution emits a stream of structured events:

```text
ResourceApplied             a record was applied
AgentCompiled               an agent record was compiled
ExecutionCreated            an execution was created
ExecutionScheduled          an execution was scheduled
ExecutionStarted            a worker began the execution
ModelCallStarted            a model call was sent
ModelCallCompleted          a model call returned
ToolCallRequested           a tool was about to be called  (AOS: steps/toolCallRequest)
ToolCallApproved            an approval was granted
ToolCallCompleted           a tool call returned          (AOS: steps/toolCallResult)
MemoryRead                  a memory store was read       (AOS: steps/memoryContextRetrieval)
MemoryWritten               a memory store was written    (AOS: steps/memoryStore)
KnowledgeRetrieved          a RAG retrieval returned      (AOS: steps/knowledgeRetrieval)
ArtifactCreated             an artifact was emitted
HookDecision                a Guardian Agent returned allow / deny / modify
PolicyDenied                a policy denied an action
ApprovalRequested           an approval was created
ApprovalGranted             an approval was granted
ApprovalDenied              an approval was denied
A2AMessageSent              an outbound A2A message was sent
A2AMessageReceived          an inbound A2A message was received
ProviderSwitched            a conversation switched providers
AgBOMUpdated                 the per-agent AgBOM was refreshed
ExecutionSucceeded          execution completed successfully
ExecutionFailed             execution failed
BudgetExceeded              a budget was exceeded
```

Each event includes `traceId`, `spanId`, `executionId`, `agentRef`, `agentVersion`, `framework`, `model`, `tenantId`, and a payload appropriate to the event type.

## OpenTelemetry mapping

Joch follows OpenTelemetry GenAI semantic conventions where they exist (`gen_ai.system`, `gen_ai.request.model`, `gen_ai.response.model`, `gen_ai.usage.input_tokens`, `gen_ai.usage.output_tokens`) and adds Joch attributes (`joch.agent.name`, `joch.framework`, `joch.policy.id`, `joch.policy.version`, `joch.tenant.id`, `joch.toolcall.id`).

See [OpenTelemetry Mapping](../../aos/opentelemetry-mapping.md).

## OCSF mapping

Security-relevant events (policy denials, approvals, A2A messages, AgBOM updates, hook decisions) emit OCSF-compatible records using `Application Activity` (8001) and `Process Activity` (1003) event classes by default.

See [OCSF Mapping](../../aos/ocsf-mapping.md).

## Sampling

```text
full         every span captured
head         capture all spans for sampled traces (e.g., 10%)
tail         capture all spans, retain only "interesting" traces (errors, denials, anomalies)
exemplar     capture exemplars per agent / per tool / per minute
```

`Trace` records reference the sampling decision so audits can see why a given execution was or was not retained.

## Retention

`retention.days` is enforced by a periodic compaction job. PII redaction can run at write or at archive.

[Back to the catalog](index.md)
