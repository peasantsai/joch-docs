# Providers

Joch routes every model call through the [Model Router](../architecture/model-router.md), which uses pluggable provider adapters. Each provider has its own page covering authentication, capabilities, region / residency, fallback, cost reporting, streaming, and known limits.

<div class="grid cards" markdown>

-   :material-creation: **OpenAI**

    ---

    `chat.completions` and `responses` APIs, structured outputs, tool calling, vision, streaming, reasoning models.

    [OpenAI provider](openai.md)

-   :material-creation: **Anthropic**

    ---

    Messages API, tool use, vision, prompt caching, extended thinking.

    [Anthropic provider](anthropic.md)

-   :material-google: **Google**

    ---

    Vertex AI / Gemini API, tool calling, multimodal, structured outputs.

    [Google provider](google.md)

-   :material-microsoft-azure: **Microsoft Foundry / Azure OpenAI**

    ---

    Azure AI Foundry projects, Azure OpenAI deployments, Entra-ID auth.

    [Microsoft Foundry provider](microsoft.md)

-   :material-server-network: **Ollama**

    ---

    Local model server with OpenAI-compatible endpoint.

    [Ollama provider](ollama.md)

-   :material-server-network: **vLLM**

    ---

    Self-hosted inference server with OpenAI-compatible API.

    [vLLM provider](vllm.md)

-   :material-server-network: **llama.cpp**

    ---

    Lightweight C++ inference server.

    [llama.cpp provider](llama-cpp.md)

</div>

## What every provider page covers

1. Authentication shape and Joch `Secret` examples.
2. Capability vector — what to declare on the [`Model`](../specs/kubernetes/model.md) record.
3. Tool calling — JSON-schema or native function-calling conventions.
4. Streaming — supported chunk types and how Joch surfaces them.
5. Region / residency — how to constrain calls.
6. Fallback behavior — when the router falls forward off this provider.
7. Cost reporting — how cost is computed per call.
8. Known limits and quirks.

## Adding a new provider

A new provider is a package, not a fork of Joch. Implementations live under `pkg/adapters/provider/<name>/`. The interface:

```ts
interface ProviderAdapter {
  capabilities(): CapabilitySet;
  respond(request: CanonicalRequest): Promise<CanonicalResponse>;
  stream(request: CanonicalRequest): AsyncIterable<CanonicalChunk>;
  health(): Promise<HealthSignal>;
}
```

A provider is supported once it passes the provider conformance suite covering tool calling, structured outputs, streaming, error mapping, and cost attribution.
