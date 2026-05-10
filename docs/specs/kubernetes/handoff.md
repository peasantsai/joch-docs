# Handoff

A `Handoff` is an explicit transfer of control between agents — the Joch resource representation of an A2A interaction. Handoffs are first-class records so multi-agent flows are auditable, replayable, and policy-governed.

[Back to the catalog](index.md)

## Spec and status

```yaml
apiVersion: joch.dev/v1alpha1
kind: Handoff
metadata:
  name: handoff-001
  namespace: support-platform
spec:
  executionRef: { name: exec-20260510-001 }

  fromAgent: { name: support-triage }
  toAgent:   { name: support-escalation }

  reason: >
    SLA at risk; escalating to support-escalation for human follow-up.

  context:
    summary: >
      Customer reported a missing refund. Confirmed eligibility under policy R-12.
      Draft reply prepared but requires senior review.
    artifactRefs:
      - artifact://exec/exec-20260510-001/draft-reply.md
    memoryRefs:
      - support-triage-working

  expectedOutput:
    type: tracked-resolution

status:
  phase: Accepted
  acceptedAt: "2026-05-10T10:36:00Z"
```

## A2A integration

Handoffs map to the A2A protocol's `send_message`, `stream_message`, `cancel_request`, `get_task`, and notification config hooks. The A2A broker enforces policy on inbound and outbound A2A messages and emits `A2AMessageSent` / `A2AMessageReceived` trace events.

## Why a separate kind

Without explicit handoff records, multi-agent flows become opaque: which agent did what, when, with what input. A `Handoff` makes the chain auditable end-to-end and policy-enforceable at the inter-agent boundary.

[Back to the catalog](index.md)
