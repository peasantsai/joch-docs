# Tool Gateway

The tool gateway is the central enforcement point for every tool call an agent makes. It is one of Joch's strongest wedges: every SDK supports tools, but none of them provides cross-SDK governance, auditing, version pinning, side-effect classification, or approvals. Joch does.

## Position in the architecture

```text
Agent (any SDK)
        │  tool call request
        ▼
[ Joch Tool Gateway ]
        │   - resolves Tool record
        │   - authenticates caller (agent identity, namespace, env)
        │   - classifies side effects
        │   - enforces Policy via Policy Engine (allow / deny / modify)
        │   - records ToolCall (idempotency key, request, approval status)
        │   - emits AOS trace events (toolCallRequest, toolCallResult)
        │   - calls the tool implementation
        │   - records result
        │   - applies output redaction / scanning per Policy
        ▼
Tool implementation (function, REST API, MCP tool, OS call)
```

Every tool call is observed, governed, and replayable.

## What the gateway resolves

Joch supports four tool sources:

```text
Function tool        a callable in user code, registered through a framework adapter
REST tool            a REST endpoint with input/output schemas
MCP tool             a tool exposed by an MCPServer (registered with the MCP Gateway)
OS tool              a sandboxed command executed by Joch (e.g., file or shell ops)
```

A [`Tool`](../specs/kubernetes/tool.md) record describes the tool's identity, schema, side-effect class, safety, observability, and gateway URL. Agents reference the record by name.

## AOS hook contract

The tool gateway implements the OWASP AOS Instrument hook contract:

```text
steps/toolCallRequest        called before the tool executes
steps/toolCallResult         called after the tool returns
```

For each call, a Guardian Agent receives the hook payload and returns one of:

| Decision | Behavior |
|---|---|
| `allow` | Execute the tool with the original arguments. |
| `deny` | Block the tool call. The agent receives a denial reason. |
| `modify` | Execute with `modifiedRequest` (e.g., redacted argument, scoped query). |

By default the Guardian role is filled by the [Policy Engine](policy-engine.md). Third-party Guardian Agents can substitute or compose.

## Side-effect classification

Every tool is tagged with a side-effect class:

```text
read_only           reads external state; no writes; no costs
local_write         writes to local agent-owned state (working memory, sandbox file)
external_write      writes to external systems (issue trackers, ticketing, Slack)
financial           moves money or commits to charges
communication       sends messages to humans (email, SMS, Slack DM)
code_execution      executes code (sandboxed REPL, browser automation)
privileged          touches sensitive infrastructure (cluster admin, DB DDL)
```

The class drives default approval requirements, default budget rules, and trace sensitivity. Operators can override per Tool record.

## Idempotency

Every `ToolCall` carries an `idempotencyKey` derived from the agent, the tool, and a stable hash of the arguments. The gateway:

- deduplicates identical tool calls within a configurable window,
- prevents duplicate side effects across [provider migrations](state-portability.md),
- offers an explicit "force re-execute" override gated by policy.

```yaml
apiVersion: joch.dev/v1alpha1
kind: ToolCall
metadata:
  name: call-abc123
spec:
  executionRef: { name: exec-20260510-001 }
  agentRef:     { name: support-triage }
  toolRef:      { name: zendesk.create_ticket }
  input:
    subject: "Refund request for order 12345"
  sideEffects:
    level: external_write
    idempotencyKey: zendesk-create-12345
  approval:
    required: true
    status: approved
    approvedBy: alice@example.com
status:
  phase: Succeeded
  startedAt:   "2026-05-10T10:35:00Z"
  completedAt: "2026-05-10T10:35:03Z"
  output:
    ticketUrl: https://example.zendesk.com/tickets/77123
```

## Approvals

Tools classified as `external_write`, `financial`, `communication`, or `privileged` can require an approval per policy. The gateway pauses the tool call, creates an [`Approval`](../specs/kubernetes/approval.md) record, and resumes (or denies) when the approver decides.

Approvers act through the CLI, the web console, Slack, email, or webhook integrations. Decisions and rationale are durable.

## Output redaction and scanning

Tool **results** are also a hook point (`toolCallResult`). The gateway can:

- scan results for prompt-injection patterns (a known MCP risk),
- redact PII before the result reaches the agent,
- truncate or summarize oversized results,
- modify the result (`modify` decision) and continue.

This is particularly important for MCP tool responses, which arrive from third-party servers and may contain attacker-controlled content.

## Rate limiting and budgets

The gateway enforces per-tool, per-agent, and per-namespace rate limits and cost budgets. A budget breach is a `BudgetExceeded` event in the trace. Hard caps deny calls; soft caps warn.

## API surface

```text
POST   /v1/tool-calls                 create + execute a tool call
GET    /v1/tool-calls/{id}            inspect a tool call
POST   /v1/tool-calls/{id}/cancel     cancel a pending tool call
POST   /v1/tool-calls/{id}/approve    approve via gateway endpoint (used by approvers)
POST   /v1/tool-calls/{id}/replay     replay (subject to policy)
```

## Operator commands

```bash
joch get toolcalls --agent support-triage --since 24h
joch describe toolcall call-abc123
joch toolcalls top --by error_rate
joch approvals ls
joch approvals decide --id appr-456 --decision approve
```

## Security considerations

- The gateway is the only place that sees raw tool arguments and results. Logs at the gateway are redacted by default.
- Secrets are resolved at the gateway boundary by the [secret broker](trust-and-security-model.md), not by the agent.
- The gateway exposes a sandboxing API for OS-class tools, with cgroups / seccomp / network egress controls.
- All denials, modifications, and approvals are immutable audit events.

The tool gateway makes the cross-cutting controls every enterprise needs uniform across SDKs. Combined with the [MCP Gateway](mcp-gateway.md), it covers nearly every external action an agent can take.
