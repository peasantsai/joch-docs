# ExecutionLoop

Kubernetes-style YAML resource specification for Joch `ExecutionLoop` resources.

[Back to Kubernetes specs](index.md)

## ExecutionLoop spec

Execution loops deserve their own resource because they define how an agent observes, reasons, acts, reflects, and stops.

```yaml
apiVersion: joch.dev/v1alpha1
kind: ExecutionLoop
metadata:
  name: react-loop
spec:
  pattern: react

  stages:
    - name: observe
    - name: think
    - name: act
    - name: reflect
    - name: decide

  maxSteps: 30

  stoppingConditions:
    - type: goal_satisfied
    - type: max_steps
    - type: budget_exceeded
    - type: approval_required
    - type: unrecoverable_error

  reflection:
    enabled: true
    everyNSteps: 5
    modelRef:
      name: gpt-5-thinking

  toolUse:
    parallelCalls: true
    maxParallelCalls: 5
    requireApprovalFor:
      - external_write
      - code_execution
      - financial

  errorHandling:
    retry:
      maxRetries: 2
      backoff: exponential
    fallback:
      switchModel: true
      switchAgent: false

  state:
    checkpointEveryNSteps: 3
    persistIntermediateThoughts: false
    persistReasoningSummary: true
```

Loop patterns we might support:

```text
react
plan_execute
reflection
debate
supervisor_worker
map_reduce
tree_search
graph_workflow
human_in_the_loop
event_driven
```

---
