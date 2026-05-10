# First Agent

The first useful Joch AI workflow is registering an existing agent without rewriting it.

## Discover agents

Run discovery against an existing repository:

```bash
joch discover ./repo
joch get agents
```

Discovery produces an `Agent` record with framework, owner, repository, runtime, models, tools, MCP servers, policies, risk level, and last execution metadata.

## Example record

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
```

## Inspect the result

```bash
joch describe agent support-triage
joch tools used-by slack.send_message
joch abom support-triage
```

The goal is visibility first. Enforcement comes after the agent, tools, and MCP servers are known.
