# Tool

`Tool` records a callable capability exposed to agents.

## Spec

```yaml
apiVersion: joch.ai/v1alpha1
kind: Tool
metadata:
  name: slack.send_message
spec:
  source:
    type: mcp
    serverRef: slack
  sideEffect: external_write
  inputSchema:
    type: object
    required: [channel, text]
    properties:
      channel:
        type: string
      text:
        type: string
  outputSchema:
    type: object
  approval:
    required: true
status:
  phase: Ready
  riskLevel: high
```

## Side-effect levels

```text
read_only
local_write
external_write
code_execution
financial
privileged
unknown
```

## Rules

- `unknown` is risky until classified.
- Tools with side effects should default to audited execution.
- `external_write`, `code_execution`, `financial`, and `privileged` should be easy to bind to approval policy.
