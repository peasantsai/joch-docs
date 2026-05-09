# Execution

Kubernetes-style YAML resource specification for Joch `Execution` resources.

[Back to Kubernetes specs](index.md)

## Execution spec

An `Execution` is one run.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Execution
metadata:
  name: exec-20260509-001
spec:
  agentRef:
    name: research-agent

  input:
    type: text
    content: >
      Research the market for agent fleet management.

  planRef:
    name: market-analysis-plan-001

  modelOverrideRef:
    name: gpt-5-thinking

  context:
    conversationId: conv_123
    memoryRefs:
      - research-agent-working-memory
    ragRefs:
      - company-docs-rag

  mode: async

  approvals:
    onToolSideEffect: pause
    onBudgetExceeded: pause

  output:
    format: markdown
    artifactName: market-analysis-report

status:
  phase: Running
  currentStep: step-2
  startedAt: "2026-05-09T10:30:00Z"
  tokenUsage:
    input: 12000
    output: 4000
  costUsd: 1.34
  toolCalls: 18
  traceRef:
    name: trace-exec-20260509-001
```

This maps well to OpenAI Agents SDK concepts like runs, tool calls, handoffs, guardrails, sessions, and tracing. ([OpenAI GitHub][2])

---

[1]: https://modelcontextprotocol.io/specification/2025-11-25?utm_source=chatgpt.com "Specification"
[2]: https://openai.github.io/openai-agents-python/tracing/?utm_source=chatgpt.com "Tracing - OpenAI Agents SDK"
[3]: https://modelcontextprotocol.io/specification/2025-11-25/server/tools?utm_source=chatgpt.com "Tools"
[4]: https://www.tomshardware.com/tech-industry/artificial-intelligence/anthropics-model-context-protocol-has-critical-security-flaw-exposed?utm_source=chatgpt.com "Anthropic's Model Context Protocol includes a critical remote code execution vulnerability - newly discovered exploit puts 200,000 AI servers at risk"
