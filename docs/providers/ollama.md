# Ollama Provider

[Ollama](https://ollama.com/) is a local model server with an OpenAI-compatible chat completion endpoint. The Joch adapter targets the `/api/chat` and `/v1/chat/completions` paths.

## Authentication

None by default. Bind to `localhost` and protect via process / network policy.

## Configuration

```yaml
apiVersion: model.joch.dev/v1alpha1
kind: Model
metadata: { name: llama-3-3-70b-local }
spec:
  provider: ollama
  model: llama-3.3-70b
  endpoint:
    type: local
    baseUrl: http://localhost:11434
  capabilities:
    text: true
    toolCalling: true
    structuredOutput: false
    streaming: true
  routing:
    regions: [on-prem]
    fallbackPolicy: on_error
```

## Capability vector

Ollama capabilities depend on the underlying model. Set `capabilities` per registered model based on what the served weights support.

## Tool calling

Many Ollama models support tool calls via JSON schemas; some do not. The adapter declares per-model capability and the router uses it for fall-forward.

## Region / residency

By definition local. Use Ollama for on-prem and EU residency-bound workloads.

## Cost reporting

Token counts are tracked but cost is reported as `0` unless you configure per-model `pricing` to reflect internal chargeback.

## Known limits

- Ollama performance varies dramatically by model and host hardware.
- Multimodal support depends on the served model.
