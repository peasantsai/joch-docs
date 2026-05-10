# ToolCall

`ToolCall` is one concrete invocation of a tool.

## Spec

```yaml
apiVersion: joch.ai/v1alpha1
kind: ToolCall
metadata:
  name: call-123
spec:
  agentRef:
    name: support-triage
  toolRef:
    name: slack.send_message
  input:
    channel: "#support"
    text: "Ticket escalated."
  sideEffect: external_write
  approvalRequired: true
status:
  phase: WaitingForApproval
```

## Phases

```text
Requested
WaitingForApproval
Denied
Approved
Running
Completed
Failed
```

## Required audit fields

```text
agent
tool
MCP server, when applicable
input summary
side-effect level
policy decision
approval reference
result summary
timestamps
```
