# Model

A `Model` resource describes a backend capability — provider, name, capabilities, limits, pricing, defaults — not just a model identifier. Joch's model router uses `Model` records together with [`ModelRoute`](model-route.md) records to make capability-aware, cost-aware, region-aware decisions.

[Back to the catalog](index.md)

## Spec

```yaml
apiVersion: joch.dev/v1alpha1
kind: Model
metadata:
  name: gpt-5-thinking
  labels:
    provider: openai
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

## Local models

Local models are first-class. Use `endpoint.type: local` and the appropriate provider adapter:

```yaml
spec:
  provider: ollama
  model: llama-3.3-70b
  endpoint:
    type: local
    baseUrl: http://localhost:11434
  auth: {}
  capabilities:
    text: true
    toolCalling: true
    structuredOutput: false
  routing:
    regions: [on-prem]
    fallbackPolicy: on_error
```

This makes data-residency-bound deployments work without changing the agent record.

## Capability matching

`capabilities` is consumed by the [model router](../../architecture/model-router.md) and by Joch's [provider migration](../../architecture/state-portability.md) checks. A request that needs vision, structured output, or long context is matched against this vector before the call is dispatched.

## Health

`status.health` is updated by passive (latency, error rate) and active (synthetic probe) signals from the model router. Routes can fall forward when health drops below threshold.

[Back to the catalog](index.md)
