# User Stories

## Agent inventory

As a platform engineer, I can run:

```bash
joch discover ./repo
joch get agents
```

So I can see agents across frameworks.

## Tool visibility

As a security engineer, I can run:

```bash
joch tools ls
joch tools used-by slack.send_message
```

So I can understand tool exposure.

## MCP governance

As a platform engineer, I can run:

```bash
joch mcp scan
joch mcp diff github --since yesterday
```

So I can detect new or changed tools.

## Policy enforcement

As a security engineer, I can define:

```yaml
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

So agents cannot perform risky actions without approval.

## Tool-call audit

As an auditor, I can run:

```bash
joch toolcalls ls --agent support-triage
joch describe toolcall call-123
```

So I can see what an agent did.

## ABOM

As a compliance lead, I can run:

```bash
joch abom support-triage
```

So I can produce an Agent Bill of Materials.
