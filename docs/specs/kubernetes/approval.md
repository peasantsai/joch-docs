# Approval

Kubernetes-style YAML resource specification for Joch `Approval` resources.

[Back to Kubernetes specs](index.md)

## Add `Approval` as a resource

Human-in-the-loop should not be bolted on.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Approval
metadata:
  name: approval-send-email-001
spec:
  requestedBy:
    executionRef:
      name: exec-20260509-001

  action:
    type: tool_call
    toolCallRef:
      name: call-send-email-001

  risk:
    level: external_write
    summary: Agent wants to send an email to a customer.

  options:
    - approve
    - reject
    - edit

  expiresAfter: 2h

status:
  phase: Pending
```

Then:

```bash
joch approvals ls
joch approvals approve approval-send-email-001
```

---
