# Agent

The `Agent` resource is the framework-agnostic record of an agent that Joch operates. The actual agent code lives in OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, CrewAI, or custom Python / TypeScript code, and is connected to Joch through a [`FrameworkAdapter`](framework-adapter.md).

[Back to the catalog](index.md)

## Spec

```yaml
apiVersion: joch.dev/v1alpha1
kind: Agent
metadata:
  name: support-triage
  namespace: support-platform
  labels:
    owner: support-platform
    env: prod
    role: triage
spec:
  displayName: Support Triage
  description: >
    Routes incoming Zendesk tickets, drafts replies, and escalates
    to human reviewers when an SLA is at risk.

  framework:
    adapterRef:
      name: openai-agents-sdk
    entrypoint: ./agents/support_triage.py
    pythonModule: support.agents.triage:agent
    image: ghcr.io/example/support-triage:1.4.0

  modelRoute:
    routeRef:
      name: research-default

  tools:
    - name: zendesk.search
    - name: zendesk.create_ticket
    - name: slack.send

  mcpServers:
    - name: github
    - name: postgres-readonly

  memories:
    working:
      name: support-triage-working
    longTerm:
      name: support-triage-long-term

  ragRefs:
    - name: support-docs-rag

  handoffs:
    allowedAgents:
      - support-escalation
      - billing-agent

  policies:
    - name: no-customer-data-exfiltration
    - name: external-send-requires-approval

  budgets:
    tokenLimitPerRun: 200000
    costLimitUsdPerDay: 25
    toolCallLimitPerRun: 100

  observability:
    tracing: enabled
    agbom: enabled
    redactSensitiveData: true

status:
  phase: Ready
  framework: openai-agents-sdk
  modelRoute: research-default
  activeExecutions: 2
  lastRunAt: "2026-05-09T10:30:00Z"
  abomRef:
    name: support-triage-agbom
```

## Identity vs. execution vs. deployment

The `Agent` record is **identity and capability**. Distinct concerns live in distinct resources:

```text
Agent          identity, capabilities, framework adapter, policies
Execution      one concrete run of the agent
Deployment     how many instances run, where, with what scaling
Conversation   the durable record of dialog state for an agent run
```

`Agent` is not a "running process." It is the configuration that produces processes through `Deployment` and runtime artifacts through `Execution`.

## Discovery

Joch can register agents from existing code:

```bash
joch discover --framework openai-agents-sdk --path ./services
joch discover --framework claude-agent-sdk --path ./coding-agents
joch discover --framework langgraph --path ./pipelines
joch discover --framework crewai --path ./marketing
```

The output is a stub `Agent` record per discovered agent. Owners review, fill in policies, model routes, and budgets, and apply.

## Status

The status subresource captures live state:

- `phase` — `Pending`, `Ready`, `Running`, `Failed`, `Suspended`.
- `framework` — the resolved framework adapter, copied from spec at compile time.
- `modelRoute` — the resolved model route used by the latest execution.
- `activeExecutions` — current concurrent runs.
- `abomRef` — pointer to the latest AgBOM. Refreshed on every change to the agent or its dependencies.

[Back to the catalog](index.md)
