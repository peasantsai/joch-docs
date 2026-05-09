# Skill

Kubernetes-style YAML resource specification for Joch `Skill` resources.

[Back to Kubernetes specs](index.md)

## Skill spec

A `Skill` should be higher-level than a tool. A skill may use multiple tools, prompts, models, and policies.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Skill
metadata:
  name: web-research
spec:
  description: Research a topic using web search and source synthesis.

  inputs:
    schema:
      type: object
      required:
        - query
      properties:
        query:
          type: string
        depth:
          type: string
          enum: [shallow, normal, deep]

  outputs:
    schema:
      type: object
      properties:
        summary:
          type: string
        sources:
          type: array
          items:
            type: string

  requiredTools:
    - web.search
    - browser.open

  requiredPolicies:
    - citation-policy
    - source-quality-policy

  execution:
    loopRef:
      name: research-loop
    maxSteps: 12

  examples:
    - input:
        query: "agent fleet management market"
        depth: normal
      output:
        summary: "..."
```

Think of the hierarchy like this:

```text
Tool = primitive callable function
Skill = reusable ability composed of tools/prompts/loops
Agent = identity that has skills
```

---
