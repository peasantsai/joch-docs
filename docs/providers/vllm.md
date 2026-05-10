# vLLM Provider

[vLLM](https://docs.vllm.ai/) is a high-throughput inference server with an OpenAI-compatible API. Used for self-hosted production inference of open-weight models.

## Configuration

```yaml
apiVersion: model.joch.dev/v1alpha1
kind: Model
metadata: { name: llama-3-1-70b-vllm }
spec:
  provider: vllm
  model: meta-llama/Llama-3.1-70B-Instruct
  endpoint:
    type: hosted
    baseUrl: http://vllm-llama:8000/v1
  auth:
    secretRef: { name: vllm-api-key }
  capabilities:
    text: true
    toolCalling: true       # set per model
    structuredOutput: true  # set per model
    streaming: true
  limits:
    contextWindowTokens: 128000
  routing:
    regions: [on-prem]
```

## Authentication

vLLM supports an optional API key (`--api-key`). When set, the adapter sends `Authorization: Bearer <key>`.

## Tool calling

Tool calling depends on the served model and the vLLM build. Declare capability per `Model` record and let the router fall forward when capability is missing.

## Streaming

Standard OpenAI SSE shape.

## Region / residency

Self-hosted; constrained by where you deploy vLLM. Use the `Model.spec.routing.regions` to enforce.

## Cost reporting

Track tokens; price as `0` or as your internal chargeback.

## Reference

- vLLM docs: <https://docs.vllm.ai/>
