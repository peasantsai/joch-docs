# Policy

`Policy` is the portable rule resource for agent action.

## Spec

```yaml
apiVersion: joch.ai/v1alpha1
kind: Policy
metadata:
  name: external-write-approval
spec:
  appliesTo:
    agents:
      selector:
        env: prod
  rules:
    - when:
        tool.sideEffect: external_write
      require:
        approval: human
    - when:
        tool.name: shell.exec
      action:
        deny
```

## Rule shape

| Field | Purpose |
|---|---|
| `when` | Conditions against agent, tool, MCP server, input, side effect, cost, or risk metadata. |
| `action` | Direct outcome such as `allow`, `deny`, or `modify`. |
| `require` | Additional gate such as `approval: human`. |

## MVP constraint

Policy v0 should be readable, explicit, and enforceable at the gateway. Avoid building a large policy language before the tool-governance wedge is proven.
