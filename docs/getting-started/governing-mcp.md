# Govern MCP

MCP gives agents a common way to reach tools and context. It also creates a new operational boundary that needs inventory, policy, approval, and audit.

## Scan servers

```bash
joch mcp scan
joch get mcpservers
joch get tools
```

Scan output is stored as `MCPServer` and `Tool` records.

## Detect drift

```bash
joch mcp diff github --since yesterday
```

Schema drift matters because a changed tool can create a changed risk profile. A read-only tool becoming an external-write tool should be visible before an agent uses it.

## Apply a policy

```yaml
apiVersion: joch.ai/v1alpha1
kind: Policy
metadata:
  name: require-approval-for-external-write
spec:
  rules:
    - when:
        tool.sideEffect: external_write
      require:
        approval: human
```

```bash
joch policy apply -f policies/require-approval-for-external-write.yaml
joch gateway start
```

## Audit calls

```bash
joch get toolcalls --agent support-triage
joch describe toolcall call-123
joch approvals ls
```

The gateway records the request, policy decision, approval state, tool result, and trace events.
