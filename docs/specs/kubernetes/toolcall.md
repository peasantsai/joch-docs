# ToolCall

Kubernetes-style YAML resource specification for Joch `ToolCall` resources.

[Back to Kubernetes specs](index.md)

## Add `ToolCall` as a runtime resource

This is easy to miss. Tool calls deserve durable records.

```yaml
apiVersion: joch.dev/v1alpha1
kind: ToolCall
metadata:
  name: call-abc123
spec:
  executionRef:
    name: exec-20260509-001

  agentRef:
    name: research-agent

  toolRef:
    name: github.create_issue

  input:
    repo: acme/joch
    title: Add Memory spec
    body: Define working, semantic, and episodic memory resources.

  sideEffects:
    level: external_write
    idempotencyKey: issue-joch-memory-spec-001

  approval:
    required: true
    status: approved
    approvedBy: alice@example.com

status:
  phase: Succeeded
  startedAt: "2026-05-09T10:35:00Z"
  completedAt: "2026-05-09T10:35:03Z"
  output:
    issueUrl: https://github.com/acme/joch/issues/123
```

Why this matters:

```text
No duplicate side effects
Auditability
Replay safety
Cost attribution
Security review
Migration across vendors
```

---
