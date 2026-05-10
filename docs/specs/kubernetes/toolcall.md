# ToolCall

A `ToolCall` is one concrete invocation of a [`Tool`](tool.md), persisted by the [tool gateway](../../architecture/tool-gateway.md). Every tool call is recorded, regardless of whether the agent is built with OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, CrewAI, or custom code.

[Back to the catalog](index.md)

## Spec and status

```yaml
apiVersion: joch.dev/v1alpha1
kind: ToolCall
metadata:
  name: call-abc123
  namespace: support-platform
spec:
  executionRef: { name: exec-20260510-001 }
  agentRef:     { name: support-triage }
  toolRef:      { name: zendesk.create_ticket }

  input:
    subject: "Refund request for order 12345"
    body: "Customer reports the item never arrived."
    priority: high

  sideEffects:
    level: external_write
    idempotent: false
    idempotencyKey: zendesk-create-12345

  approval:
    required: true
    status: approved
    approvedBy: alice@example.com
    approvedAt: "2026-05-10T10:34:55Z"

  hookContext:
    aosTurnId: 69ef57b8-3993-440d-9493-523914f3f149
    aosStepId: 9263448a-186a-4c3b-abcf-443feb44a01e

status:
  phase: Succeeded
  startedAt: "2026-05-10T10:35:00Z"
  completedAt: "2026-05-10T10:35:03Z"
  output:
    ticketUrl: https://example.zendesk.com/tickets/77123
    ticketId: 77123
  hookDecisions:
    - hook: steps/toolCallRequest
      decision: allow
      policyId: external-send-requires-approval
      policyVersion: v3
    - hook: steps/toolCallResult
      decision: allow
      policyId: external-send-requires-approval
      policyVersion: v3
```

## Why this matters

Without durable `ToolCall` records:

- side effects can repeat after retries or migrations,
- audits cannot prove what an agent did,
- cost cannot be attributed to specific tools,
- approvals cannot be reconstructed,
- replay is impossible.

The `ToolCall` is therefore one of the most valuable runtime resources in the catalog.

## AOS hook integration

Every tool call corresponds to two AOS hook events:

- `steps/toolCallRequest` (before the call), and
- `steps/toolCallResult` (after the call).

The hook decisions are recorded in `status.hookDecisions` so an audit can see which policy version produced which outcome — `allow`, `deny`, or `modify` — and why.

## Idempotency

`sideEffects.idempotencyKey` is honored by the tool gateway. Within a configurable window, identical keys collapse to a single execution; subsequent attempts return the original result without re-executing the tool.

## Approval

When `approval.required` is true, the gateway pauses the call, creates an [`Approval`](approval.md), and resumes (or denies) on decision. The approver, decision, and rationale are durable.

## Replay

```bash
joch describe toolcall call-abc123
joch toolcalls replay call-abc123 --policy strict
```

Replay is gated by policy. Idempotent reads can replay freely; non-idempotent writes require explicit override.

[Back to the catalog](index.md)
