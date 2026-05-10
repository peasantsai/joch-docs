# Agent

`Agent` is the framework-agnostic record of an agent that Joch AI inventories and governs. The actual loop stays in the agent framework.

## Spec

```yaml
apiVersion: joch.ai/v1alpha1
kind: Agent
metadata:
  name: support-triage
  namespace: default
  labels:
    env: prod
    team: support
spec:
  framework:
    type: crewai
    entrypoint: ./agents/support_triage.py
  owner:
    team: support-platform
  repository: https://github.com/example/support-agents
  runtime:
    type: docker
    image: ghcr.io/example/support-triage:0.1.0
  models:
    - openai:gpt-5
  tools:
    - zendesk.search
    - zendesk.update_ticket
    - slack.send_message
  mcpServers:
    - zendesk
    - slack
  policies:
    - external-write-approval
status:
  phase: Registered
  riskLevel: high
  lastExecution: exec-123
```

## Required behavior

- Discovery can create stub `Agent` records.
- Owners can enrich the record with labels, ownership, policy refs, and risk context.
- ABOM generation starts from the `Agent` record.
- Gateway audit records must reference the calling `Agent`.
