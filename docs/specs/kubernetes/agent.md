# Agent

Kubernetes-style YAML resource specification for Joch `Agent` resources.

[Back to Kubernetes specs](index.md)

## Agent spec

An `Agent` is the identity and behavior contract. It should not be tied to one model provider.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Agent
metadata:
  name: research-agent
  namespace: default
  labels:
    role: researcher
    tier: production
spec:
  displayName: Research Agent

  description: >
    Performs market research, summarizes findings, and creates structured reports.

  modelRef:
    name: gpt-5-thinking
    fallbackRefs:
      - claude-sonnet
      - gemini-pro

  personalityRef:
    name: pragmatic-researcher

  systemPromptRef:
    name: research-agent-system

  skills:
    - name: web-research
    - name: summarize-documents
    - name: cite-sources

  tools:
    - name: web.search
    - name: browser.open
    - name: filesystem.write

  mcpServers:
    - name: github
    - name: slack
    - name: postgres-readonly

  memories:
    working:
      name: research-agent-working-memory
    longTerm:
      name: company-research-memory
    episodic:
      name: research-agent-episodes

  ragRefs:
    - name: company-docs-rag
    - name: market-reports-rag

  planning:
    mode: hierarchical
    plannerRef:
      name: default-planner
    maxPlanDepth: 5
    requirePlanApproval: false

  execution:
    loopRef:
      name: react-loop
    maxSteps: 30
    timeout: 30m
    retryPolicy:
      maxRetries: 2
      backoff: exponential

  handoffs:
    allowedAgents:
      - writer-agent
      - data-agent
      - reviewer-agent
    strategy: explicit

  guardrails:
    input:
      - name: pii-filter
    output:
      - name: citation-required
      - name: no-confidential-leakage
    tool:
      - name: dangerous-tool-approval

  policies:
    - name: research-agent-policy

  budgets:
    tokenLimitPerRun: 200000
    costLimitUsdPerDay: 25
    toolCallLimitPerRun: 100

  observability:
    tracing: enabled
    logLevel: info
    redactSensitiveData: true

status:
  phase: Ready
  activeExecutions: 2
  lastRunAt: "2026-05-09T10:30:00Z"
  currentModel: gpt-5-thinking
```

Important distinction:

```text
Agent = identity, capabilities, behavior, permissions
Execution = one concrete run of that agent
Deployment = how many instances / where / scaling / rollout
```

Do **not** make `Agent` equal to “running process.”

---
