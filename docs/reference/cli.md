# CLI Reference

The `joch` CLI is the primary operator surface.

## MVP commands

```bash
joch init
joch up
joch apply -f .
joch discover ./repo
joch get agents
joch get tools
joch get mcpservers
joch get toolcalls
joch get approvals
joch abom <agent>
joch policy apply -f policy.yaml
joch gateway start
joch trace <execution>
```

## Discovery

```bash
joch discover ./repo
joch get agents
joch describe agent support-triage
```

## Tools

```bash
joch tools ls
joch tools used-by slack.send_message
joch get toolcalls --agent support-triage
joch describe toolcall call-123
```

## MCP

```bash
joch mcp scan
joch mcp diff github --since yesterday
joch get mcpservers
joch describe mcpserver github
```

## Policy

```bash
joch policy apply -f policies/no-external-write.yaml
joch get policies
joch describe policy external-write-approval
```

## Approvals

```bash
joch approvals ls
joch approvals approve approval-123
joch approvals reject approval-123
```

## ABOM and trace

```bash
joch abom support-triage
joch trace exec-123
```

## Output flags

```bash
joch get agents -o table
joch get agents -o yaml
joch get toolcalls -o json
```
