# Plan

Kubernetes-style YAML resource specification for Joch `Plan` resources.

[Back to Kubernetes specs](index.md)

## Plan spec

A `Plan` is the proposed route to accomplish a goal.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Plan
metadata:
  name: market-analysis-plan-001
spec:
  goal: >
    Analyze the market opportunity for joch.

  owner:
    kind: Agent
    name: research-agent

  planner:
    strategy: hierarchical
    modelRef:
      name: gpt-5-thinking

  steps:
    - id: step-1
      title: Identify competing tools
      assignedAgent: research-agent
      requiredSkills:
        - web-research
      successCriteria:
        - At least 10 relevant tools identified

    - id: step-2
      title: Compare positioning
      assignedAgent: analyst-agent
      dependsOn:
        - step-1
      successCriteria:
        - Differentiation table produced

    - id: step-3
      title: Draft recommendation
      assignedAgent: writer-agent
      dependsOn:
        - step-2

  approvals:
    requiredBeforeExecution: false
    requiredBeforeExternalWrite: true

  constraints:
    maxDuration: 2h
    maxCostUsd: 10
    requireCitations: true

status:
  phase: Approved
  progress:
    completedSteps: 0
    totalSteps: 3
```

Plans should be versioned. Agents will revise plans.

```yaml
status:
  currentRevision: 3
  previousRevisions:
    - 1
    - 2
```

---
