# Trace

Kubernetes-style YAML resource specification for Joch `Trace` resources.

[Back to Kubernetes specs](index.md)

## Trace spec

Do not treat logs as an afterthought. Modern agent SDKs increasingly include tracing of LLM generations, tool calls, handoffs, guardrails, and custom events. ([OpenAI GitHub][2])

```yaml
apiVersion: joch.dev/v1alpha1
kind: Trace
metadata:
  name: trace-exec-20260509-001
spec:
  executionRef:
    name: exec-20260509-001

  sampling:
    mode: full

  export:
    openTelemetry:
      enabled: true
      endpointSecretRef:
        name: otel-collector

  retention:
    days: 30

status:
  phase: Complete
  spans:
    total: 84
    llmCalls: 9
    toolCalls: 18
    handoffs: 2
    guardrailChecks: 6
```

Joch may also need `Event` as a lower-level append-only primitive.

---

[1]: https://modelcontextprotocol.io/specification/2025-11-25?utm_source=chatgpt.com "Specification"
[2]: https://openai.github.io/openai-agents-python/tracing/?utm_source=chatgpt.com "Tracing - OpenAI Agents SDK"
[3]: https://modelcontextprotocol.io/specification/2025-11-25/server/tools?utm_source=chatgpt.com "Tools"
[4]: https://www.tomshardware.com/tech-industry/artificial-intelligence/anthropics-model-context-protocol-has-critical-security-flaw-exposed?utm_source=chatgpt.com "Anthropic's Model Context Protocol includes a critical remote code execution vulnerability - newly discovered exploit puts 200,000 AI servers at risk"
