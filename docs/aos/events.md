# Events (Trace)

The **trace** pillar of OWASP AOS specifies that agents emit comprehensive events on every runtime decision and lifecycle change. Joch implements this through the [`Trace`](../specs/kubernetes/trace.md) resource and a structured event taxonomy that extends OpenTelemetry and OCSF.

## Why a unified event log

Without a unified event log, multi-step agent reasoning is opaque. Operators cannot tell which model produced which output, which tool result fed which decision, or which Guardian decision blocked which side effect. The unified log answers all of those questions in one place.

## Event taxonomy

Joch emits the following event types. Each carries a common envelope and an event-specific payload.

```text
ResourceApplied             a record was applied
AgentCompiled               an agent record was compiled into a manifest
ExecutionCreated            an execution was created
ExecutionScheduled          an execution was assigned to a worker
ExecutionStarted            a worker began execution
ModelCallStarted            a model call was sent to the provider
ModelCallCompleted          a model call returned
ToolCallRequested           a tool was about to be called   (AOS hook: steps/toolCallRequest)
ToolCallApproved            an approval was granted
ToolCallCompleted           a tool call returned            (AOS hook: steps/toolCallResult)
MemoryRead                  a memory store was read         (AOS hook: steps/memoryContextRetrieval)
MemoryWritten               a memory store was written      (AOS hook: steps/memoryStore)
KnowledgeRetrieved          a RAG retrieval returned        (AOS hook: steps/knowledgeRetrieval)
ArtifactCreated             a durable artifact was emitted
HookDecision                a Guardian Agent returned allow / deny / modify
PolicyDenied                a policy denied an action
ApprovalRequested           an approval was created
ApprovalGranted             an approval was granted
ApprovalDenied              an approval was denied
A2AMessageSent              an outbound A2A message was sent
A2AMessageReceived          an inbound A2A message was received
ProviderSwitched            a conversation switched providers
AgBOMUpdated                 the per-agent AgBOM was refreshed
ExecutionSucceeded          an execution completed successfully
ExecutionFailed             an execution failed
BudgetExceeded              a budget was exceeded
```

## Common envelope

Every event includes:

```text
traceId                    OpenTelemetry trace id
spanId                     OpenTelemetry span id
parentSpanId               (optional) parent span id
executionId                Joch Execution name
agentRef                   Agent name + namespace
agentVersion               agent record version
framework                  framework adapter (e.g., openai-agents-sdk)
modelRef                   model record name (when applicable)
tenantId                   namespace + team
turnId                     AOS turn id
stepId                     AOS step id
timestamp                  ISO 8601 UTC
```

The envelope makes events joinable across spans, agents, and tenants without bespoke processing.

## Standard mappings

Joch does not invent a new schema:

- **OpenTelemetry**: spans use OTel GenAI semantic conventions (`gen_ai.*`) and add `joch.*` attributes for agent-specific context. See [OpenTelemetry Mapping](opentelemetry-mapping.md).
- **OCSF**: security-relevant events emit OCSF-compatible records under `Application Activity` (8001) and `Process Activity` (1003) by default. See [OCSF Mapping](ocsf-mapping.md).

You can ship Joch traces into any backend that consumes OTLP and OCSF — Grafana, Honeycomb, Datadog, Splunk, Elastic, your SIEM.

## Sampling and retention

The [`Trace`](../specs/kubernetes/trace.md) resource controls sampling and retention per execution or per agent:

```text
full         every span captured
head         capture all spans for sampled traces (e.g., 10%)
tail         capture all spans, retain only "interesting" traces
exemplar     capture exemplars per agent / per tool / per minute
```

Retention is a configurable number of days, with optional pre-archive PII redaction.

## Why this matters for AOS conformance

The combination of:

- standard hooks ([Hooks](hooks.md)),
- standard events (this page),
- and standard inventory ([AgBOM](agbom.md))

is the AOS contract. Implementing all three faithfully — and exporting through industry-standard transports — is what makes Joch agents interoperable with third-party Guardian Agents, BOM consumers, and trace backends without bespoke integration.
