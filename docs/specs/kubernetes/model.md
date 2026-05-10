# Model

Kubernetes-style YAML resource specification for Joch `Model` resources.

[Back to Kubernetes specs](index.md)

## Model spec

A `Model` should describe a backend capability, not just a model name.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Model
metadata:
  name: gpt-5-thinking
spec:
  provider: openai
  model: gpt-5.5-thinking

  endpoint:
    type: hosted
    baseUrlSecretRef:
      name: openai-endpoint
      key: baseUrl

  auth:
    secretRef:
      name: openai-api-key

  capabilities:
    text: true
    vision: true
    audio: false
    toolCalling: true
    structuredOutput: true
    jsonSchema: true
    streaming: true
    reasoning: true
    computerUse: false
    embeddings: false

  limits:
    contextWindowTokens: 400000
    maxOutputTokens: 64000
    requestsPerMinute: 100
    tokensPerMinute: 2000000

  pricing:
    currency: USD
    inputPerMillionTokens: 0
    outputPerMillionTokens: 0

  defaultParameters:
    temperature: 0.3
    topP: 1
    reasoningEffort: medium

  routing:
    priority: 100
    regions:
      - eu
      - us
    fallbackPolicy: on_error_or_budget

status:
  phase: Ready
  health: Healthy
  latencyP50Ms: 900
  latencyP95Ms: 2800
```

Joch will need model capability matching for commands like:

```bash
joch run research-agent --model claude-sonnet
joch models check --agent research-agent --target claude-sonnet
```

The compatibility layer should answer:

```text
Can this model run this agent with its tools, memory, planning loop, and output constraints?
```

---
