# Prompt

Kubernetes-style YAML resource specification for Joch `Prompt` resources.

[Back to Kubernetes specs](index.md)

## Prompt spec

MCP has a prompt primitive for templated messages/workflows, so `joch` should have a native `Prompt` resource too. ([Model Context Protocol][1])

```yaml
apiVersion: joch.dev/v1alpha1
kind: Prompt
metadata:
  name: research-agent-system
spec:
  type: system
  templateEngine: handlebars

  template: |
    we are {{agent.displayName}}.
    our task is to help with {{task.domain}}.
    Follow these policies:
    {{#each policies}}
    - {{this}}
    {{/each}}

  variables:
    - name: task.domain
      required: true
    - name: policies
      required: false

  outputContract:
    format: markdown
    requireCitations: true
```

we may also want:

```text
PromptPack
```

for reusable prompt bundles.

---

[1]: https://modelcontextprotocol.io/specification/2025-11-25?utm_source=chatgpt.com "Specification"
[2]: https://openai.github.io/openai-agents-python/tracing/?utm_source=chatgpt.com "Tracing - OpenAI Agents SDK"
[3]: https://modelcontextprotocol.io/specification/2025-11-25/server/tools?utm_source=chatgpt.com "Tools"
[4]: https://www.tomshardware.com/tech-industry/artificial-intelligence/anthropics-model-context-protocol-has-critical-security-flaw-exposed?utm_source=chatgpt.com "Anthropic's Model Context Protocol includes a critical remote code execution vulnerability - newly discovered exploit puts 200,000 AI servers at risk"
