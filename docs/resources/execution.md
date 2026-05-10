# Execution

`Execution` records one concrete run of an agent.

## Spec

```yaml
apiVersion: joch.ai/v1alpha1
kind: Execution
metadata:
  name: exec-123
spec:
  agentRef:
    name: support-triage
  trigger:
    type: manual
  inputSummary: Triage incoming support queue.
status:
  phase: Running
  startedAt: "2026-05-11T00:00:00Z"
```

## Links

An execution links together:

```text
agent
tool calls
approvals
trace events
ABOM generation, when triggered
```

The MVP does not need to own the agent loop, but it does need a durable execution record for audit.
