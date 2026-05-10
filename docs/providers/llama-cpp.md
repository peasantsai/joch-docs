# llama.cpp Provider

[llama.cpp](https://github.com/ggerganov/llama.cpp) is a lightweight C++ inference server. The HTTP server (`llama-server`) exposes an OpenAI-compatible API.

## Configuration

```yaml
apiVersion: model.joch.dev/v1alpha1
kind: Model
metadata: { name: llama-3-1-8b-llamacpp }
spec:
  provider: llama-cpp
  model: llama-3.1-8b-instruct.Q4_K_M.gguf
  endpoint:
    type: local
    baseUrl: http://localhost:8080/v1
  capabilities:
    text: true
    toolCalling: false   # depends on model + build
    structuredOutput: false
    streaming: true
  routing:
    regions: [on-prem]
```

## Authentication

Optional bearer token via `--api-key`.

## When to use

- Resource-constrained edge / on-prem deployments.
- Offline workloads where Ollama or vLLM is too heavy.
- Development with quantized models on CPU or modest GPUs.

## Limits

- Tool calling support is uneven and depends on chat templates compiled into the build.
- Throughput is significantly lower than vLLM for batch workloads.

## Reference

- llama.cpp HTTP server: <https://github.com/ggerganov/llama.cpp/blob/master/examples/server/README.md>
