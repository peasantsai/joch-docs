# TraceEvent

`TraceEvent` is a durable event emitted by discovery, gateway, policy, approval, execution, and ABOM workflows.

## Spec

```yaml
apiVersion: joch.ai/v1alpha1
kind: TraceEvent
metadata:
  name: event-123
spec:
  type: toolcall.requested
  agentRef:
    name: support-triage
  executionRef:
    name: exec-123
  toolCallRef:
    name: call-123
  timestamp: "2026-05-11T00:00:00Z"
  data:
    tool: slack.send_message
    sideEffect: external_write
status:
  phase: Recorded
```

## Event types

```text
agent.discovered
tool.discovered
mcp.scanned
policy.applied
toolcall.requested
toolcall.approved
toolcall.denied
toolcall.completed
execution.started
execution.completed
abom.generated
```

Trace events are the foundation for audit exports, dashboards, and incident reconstruction.
