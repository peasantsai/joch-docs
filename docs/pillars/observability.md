# Observability

> Trace every model call, tool call, memory write, RAG retrieval, approval, cost line item, and artifact — and surface the operator views that vendor SDKs do not.

Observability is what powers governance, release management, cost control, and incident response. Generic tracing is not enough — Joch ships **agent operations views** in the language of agent fleets.

## Trace events

Every Joch execution emits a structured event stream that extends OpenTelemetry and OCSF, in alignment with the [OWASP AOS Trace specification](../aos/events.md):

```text
ResourceApplied             a record was applied
AgentCompiled               an agent record was compiled into a manifest
ExecutionCreated            an execution was created
ExecutionScheduled          an execution was assigned to a worker
ExecutionStarted            a worker began execution
ModelCallStarted            a model call was sent to the provider
ModelCallCompleted          a model call returned
ToolCallRequested           a tool was about to be called (AOS hook: toolCallRequest)
ToolCallApproved            an approval was granted
ToolCallCompleted           a tool call returned (AOS hook: toolCallResult)
MemoryRead                  a memory store was read (AOS hook: memoryContextRetrieval)
MemoryWritten               a memory store was written (AOS hook: memoryStore)
KnowledgeRetrieved          a RAG retrieval returned (AOS hook: knowledgeRetrieval)
ArtifactCreated             a durable artifact was emitted
HookDecision                a Guardian Agent returned allow / deny / modify
PolicyDenied                a policy denied an action
ApprovalRequested           an approval was created
ApprovalGranted             an approval was granted
ApprovalDenied              an approval was denied
A2AMessageSent              an outbound A2A message was sent
A2AMessageReceived          an inbound A2A message was received
ExecutionSucceeded          an execution completed successfully
ExecutionFailed             an execution failed
BudgetExceeded              a cost / usage budget was exceeded
ProviderSwitched            a conversation switched providers
ABOMUpdated                 the per-agent ABOM was refreshed
```

Every event includes `traceId`, `spanId`, `executionId`, `agentRef`, `agentVersion`, `framework`, `model`, `tenantId`, and a payload appropriate to the event type.

## OpenTelemetry and OCSF

Joch trace events extend industry-standard schemas, not proprietary ones:

- **OpenTelemetry** — Spans use OTel semantic conventions where they exist (`gen_ai.system`, `gen_ai.request.model`, `gen_ai.response.model`, etc.) and add Joch-specific attributes (`joch.agent.name`, `joch.framework`, `joch.policy.id`, `joch.tenant.id`). See [OpenTelemetry Mapping](../aos/opentelemetry-mapping.md).
- **OCSF** — Security-relevant events (policy denial, approval, A2A messages, ABOM updates) emit OCSF-compatible records. See [OCSF Mapping](../aos/ocsf-mapping.md).

You can ship Joch traces into Grafana, Honeycomb, Datadog, Splunk, Elastic, your SIEM, or any backend that consumes OTLP and OCSF.

## Operator views

Generic tracing answers "what happened in this span." Joch additionally answers operator-language questions:

```bash
joch top agents                    # most active agents by execution count / cost
joch top tools                     # most-called tools, with success rate
joch top models                    # most-used models, by cost and latency
joch cost by-team                  # cost rollups per team / namespace
joch cost by-agent --since 7d
joch trace exec-123                # full execution trace
joch incidents ls                  # current incidents flagged by alerts
joch drift detect --agent X        # output drift after a deployment
joch denials ls --policy P         # recent policy denials
joch approvals ls                  # pending approvals
```

The web console exposes the same views with charts, filters, and drill-downs.

## Cost accounting

Cost is a first-class observability primitive. Every model call, tool call, and approval has an attributed cost. Joch rolls up cost per:

```text
Agent
Execution
Conversation
Tool
Model
Team
Environment
Time window
```

Costs feed [`Budget`](../specs/kubernetes/budget.md) enforcement. A budget breach can either alert, soft-cap, or hard-deny depending on policy.

## Quality observability

Beyond cost and latency, Joch tracks **agent quality**:

- Tool failure rate per tool, per agent, per environment.
- Model fallback rate (how often `ModelRoute` fell off the primary).
- RAG retrieval quality scores (top-k, judge-scored relevance, citation rate).
- Memory write volume and growth.
- Guardrail hit rate (AOS hook decisions: allow vs. modify vs. deny).
- Approval bottleneck (queue depth, time-to-decision).
- Prompt / version change correlation with regression.

These metrics feed the [Release Management pillar](release-management.md).

## Audit trail

Every decision is replayable. Given an execution ID, an operator can reconstruct:

```text
inputs
retrieved knowledge (with citations)
memory reads
model calls (prompts, responses, tokens)
tool calls (args, side-effect class, approval, result)
memory writes
hook decisions (allow / deny / modify, by rule and policy version)
artifacts produced
costs charged
provider switches
A2A messages
final outputs
```

This is the foundation of compliance: every action has a who, when, what, why, and the policy version active at the time. See [`Trace`](../specs/kubernetes/trace.md).

## Acceptance criteria

A team operating Joch's Observability pillar can:

- replay any execution from inputs to outputs, with policy version, cost, latency, and hook decisions,
- spot a tool failure rate spike within minutes and trace it to a specific MCP server version,
- prove that no `email.send` call left the gateway in the last 90 days without a recorded approval,
- export the full trace stream to OpenTelemetry and the security-relevant subset to OCSF.
