# ABOM

`ABOM` is the Agent Bill of Materials for a registered agent.

## Spec

```yaml
apiVersion: joch.ai/v1alpha1
kind: ABOM
metadata:
  name: support-triage-abom
spec:
  agentRef:
    name: support-triage
  framework: crewai
  models:
    - openai:gpt-5
  tools:
    - slack.send_message
    - zendesk.update_ticket
  mcpServers:
    - slack
    - zendesk
  memories:
    - support-working-memory
  ragSources:
    - support-docs
  secretsReferenced:
    - zendesk-api-token
  policies:
    - external-write-approval
  deploymentTargets:
    - docker-compose
    - kubernetes
  risk:
    level: high
    reasons:
      - can_send_external_messages
      - can_access_customer_data
```

## Commands

```bash
joch abom support-triage
```

## MVP rule

ABOM v0 should be generated from real registry records. A schema row that no code populates is not a feature.
