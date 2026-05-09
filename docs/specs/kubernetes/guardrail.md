# Guardrail

Kubernetes-style YAML resource specification for Joch `Guardrail` resources.

[Back to Kubernetes specs](index.md)

## Guardrail spec

Policy is broad governance. Guardrails are runtime checks.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Guardrail
metadata:
  name: citation-required
spec:
  type: output

  check:
    kind: llm_judge
    modelRef:
      name: gpt-5-mini
    criteria:
      - Every factual claim from external sources must include a citation.
      - Do not cite sources that were not actually used.

  action:
    onFail: retry
    maxRetries: 1
    fallback: block

status:
  phase: Ready
```

Guardrail types:

```text
input
output
tool
memory_write
rag_retrieval
handoff
budget
security
```

---
