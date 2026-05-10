# Approval

`Approval` records a human decision for a risky action.

## Spec

```yaml
apiVersion: joch.ai/v1alpha1
kind: Approval
metadata:
  name: approval-123
spec:
  toolCallRef:
    name: call-123
  policyRef:
    name: external-write-approval
  requestedAction:
    type: tool_call
    summary: Send a Slack message to #support.
  risk:
    level: external_write
  options:
    - approve
    - reject
status:
  phase: Pending
```

## Commands

```bash
joch approvals ls
joch approvals approve approval-123
joch approvals reject approval-123
```

## Required decision fields

```text
decision
decidedBy
decidedAt
rationale
policyRef
toolCallRef
```

Approvals must be queryable independently of tool calls so auditors can inspect human decisions directly.
