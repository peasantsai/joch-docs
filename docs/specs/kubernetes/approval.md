# Approval

An `Approval` is a human-review record. Approvals gate side-effecting tool calls, release promotions, and emergency policy overrides. Decisions are durable and feed the audit log.

[Back to the catalog](index.md)

## Spec and status

```yaml
apiVersion: joch.dev/v1alpha1
kind: Approval
metadata:
  name: approval-call-abc123
  namespace: support-platform
spec:
  requestedBy:
    executionRef: { name: exec-20260510-001 }

  action:
    type: tool_call
    toolCallRef: { name: call-abc123 }

  policyRef:
    name: external-send-requires-approval

  risk:
    level: external_write
    summary: Agent wants to create a Zendesk ticket on behalf of a customer.

  routing:
    approvers:
      - team:support-platform-leads
    timeoutMinutes: 30
    onTimeout: deny

  options:
    - approve
    - reject
    - edit

  channels:
    - slack:#support-approvals
    - email:support-platform-leads@example.com

status:
  phase: Approved
  decision: approve
  decidedBy: alice@example.com
  decidedAt: "2026-05-10T10:34:55Z"
  rationale: "Refund eligibility confirmed."
```

## Action types

```text
tool_call       approve a specific ToolCall
promotion       approve a Deployment / agent / policy promote
override        approve a break-glass policy override
```

## Operator workflow

```bash
joch approvals ls
joch describe approval approval-call-abc123
joch approvals decide approval-call-abc123 --decision approve --rationale "Refund eligibility confirmed."
joch approvals decide approval-release-001    --decision reject  --rationale "Eval below threshold."
```

The web console and Slack / email integrations surface the same actions.

## Why a separate kind

Approvals are an audit-class primitive: they must be durable, replayable, and bound to the action that triggered them. Embedding approval state inside `ToolCall` or `Deployment` would make decisions hard to query and impossible to audit independently.

[Back to the catalog](index.md)
