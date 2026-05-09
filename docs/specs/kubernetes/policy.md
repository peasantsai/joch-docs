# Policy

Kubernetes-style YAML resource specification for Joch `Policy` resources.

[Back to Kubernetes specs](index.md)

## Policy spec

Policies are critical for enterprise adoption.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Policy
metadata:
  name: research-agent-policy
spec:
  appliesTo:
    agents:
      - research-agent

  modelPolicy:
    allowedProviders:
      - openai
      - anthropic
    deniedModels:
      - experimental-unsafe-model

  dataPolicy:
    allowExternalDataTransfer: false
    pii:
      action: redact
    retention:
      maxDays: 30

  toolPolicy:
    default: deny
    allow:
      - web.search
      - browser.open
      - filesystem.write
    requireApproval:
      - github.create_issue
      - email.send
      - shell.exec

  networkPolicy:
    allowedDomains:
      - wikipedia.org
      - sec.gov
      - github.com

  budgetPolicy:
    maxCostUsdPerRun: 5
    maxCostUsdPerDay: 50

  audit:
    logPrompts: true
    logToolCalls: true
    logOutputs: true
    redactSecrets: true
```

Later, this could compile to OPA/Rego or Cedar-style policies.

---
