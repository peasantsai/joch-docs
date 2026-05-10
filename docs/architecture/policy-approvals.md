# Policy and Approvals

Policy v0 focuses on tool and MCP risk. It is intentionally simple enough for operators to read and strict enough to enforce before side effects.

## Policy model

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

## Enforcement points

```text
MCP gateway       before MCP tools/call and after results
Tool gateway      before generic tools and after results
Worker scans      when discovered capabilities change
CLI/API apply     when policies are validated
```

## Decision outcomes

```text
allow      continue the call
deny       reject the call and record the policy reason
require    create an approval and pause the call
modify     redact or transform the request/result before continuing
```

## Approval workflow

```bash
joch approvals ls
joch approvals approve approval-123
joch approvals reject approval-123
```

An approval record must include:

```text
requested action
agent
tool
input summary
policy that required approval
risk level
decision
decider
rationale
timestamp
```

## Non-goals for v0

```text
complex policy compilation
multi-party approval quorums
advanced risk scoring
full compliance workflow automation
```

Those can build on the same records later.
