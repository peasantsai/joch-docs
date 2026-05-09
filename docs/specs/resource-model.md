# Resource Model

## My initial answer

I would solve OpenAI-to-Claude mid-conversation switching with:

```text
Canonical state store
+ event-sourced conversation log
+ normalized tool-call records
+ externalized memory/artifacts
+ migration checkpoints
+ provider adapters
+ capability validation
```

The most important architectural decision is this:

> `joch` should own agent identity, memory, and lifecycle.
> Vendors should only provide inference backends.

That is the difference between a real fleet manager and a thin wrapper around SDKs.

I would make `joch` specs feel like **Kubernetes CRDs for agent systems**.

The core idea:

```yaml
apiVersion: joch.dev/v1alpha1
kind: Agent
metadata:
  name: research-agent
  labels:
    team: growth
    env: dev
spec:
  ...
status:
  ...
```

Every resource should have:

```yaml
apiVersion: joch.dev/v1alpha1
kind: <ResourceKind>
metadata:
  name: string
  namespace: string
  labels: {}
  annotations: {}
spec: {}
status:
  phase: Pending | Ready | Running | Failed | Suspended
  conditions: []
  observedGeneration: number
```

This gives we a common control-plane model for agents, models, memories, tools, deployments, and runtime state.

MCP should strongly influence the design: its official primitives include **tools**, **resources**, and **prompts**, with tools exposing callable functions and resources exposing context/data by URI. ([Model Context Protocol][1]) OpenAI’s Agents SDK also treats tools, handoffs, guardrails, tracing, and sessions as first-class orchestration concepts, which map well into `joch` resources. ([OpenAI GitHub][2])

---

# 1. Top-level resource kinds

I would start with these:

```text
Agent
Model
Skill
Tool
MCPServer
Personality
Memory
RAG
KnowledgeSource
Prompt
Plan
Execution
Deployment
Policy
Guardrail
Secret
Budget
Eval
Trace
Artifact
Environment
Team
```

The minimum useful v1 should include:

```text
Agent
Model
Skill
MCPServer
Personality
Memory
RAG
Plan
Execution
Deployment
Policy
Trace
```

The rest can come later.

---

# 2. Agent spec

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

# 3. Model spec

A `Model` should describe a backend capability, not just a model name.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Model
metadata:
  name: gpt-5-thinking
spec:
  provider: openai
  model: gpt-5.5-thinking

  endpoint:
    type: hosted
    baseUrlSecretRef:
      name: openai-endpoint
      key: baseUrl

  auth:
    secretRef:
      name: openai-api-key

  capabilities:
    text: true
    vision: true
    audio: false
    toolCalling: true
    structuredOutput: true
    jsonSchema: true
    streaming: true
    reasoning: true
    computerUse: false
    embeddings: false

  limits:
    contextWindowTokens: 400000
    maxOutputTokens: 64000
    requestsPerMinute: 100
    tokensPerMinute: 2000000

  pricing:
    currency: USD
    inputPerMillionTokens: 0
    outputPerMillionTokens: 0

  defaultParameters:
    temperature: 0.3
    topP: 1
    reasoningEffort: medium

  routing:
    priority: 100
    regions:
      - eu
      - us
    fallbackPolicy: on_error_or_budget

status:
  phase: Ready
  health: Healthy
  latencyP50Ms: 900
  latencyP95Ms: 2800
```

we will want model capability matching for commands like:

```bash
joch run research-agent --model claude-sonnet
joch models check --agent research-agent --target claude-sonnet
```

The compatibility layer should answer:

```text
Can this model run this agent with its tools, memory, planning loop, and output constraints?
```

---

# 4. Personality spec

This is one of the strongest differentiators. Treat personality as code.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Personality
metadata:
  name: pragmatic-researcher
spec:
  version: 1

  files:
    heartbeat: ./personality/heartbeat.md
    soul: ./personality/soul.md
    style: ./personality/style.md
    boundaries: ./personality/boundaries.md

  traits:
    tone: concise
    reasoningStyle: evidence-first
    riskTolerance: low
    creativity: medium
    autonomy: medium

  behavioralRules:
    - Always cite external factual claims.
    - Prefer structured summaries.
    - Ask clarifying questions only when execution would otherwise be unsafe or materially wrong.
    - Do not fabricate sources.

  communication:
    verbosity: medium
    formatPreference: markdown
    defaultAudience: technical-founder

  identity:
    stableName: research-agent
    role: Market and technical research assistant
    doNotImpersonateHuman: true

status:
  phase: Ready
  checksum: sha256:abc123
```

I would separate:

```text
Personality = how the agent behaves
Prompt = task/system instruction content
Policy = what the agent is allowed to do
Memory = what the agent remembers
```

Do not overload one giant system prompt with all four.

---

# 5. Prompt spec

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

# 6. Skill spec

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

# 7. Tool spec

Tools are callable functions. MCP tools are the obvious compatibility target: MCP tools have names and input schemas and let models interact with external systems. ([Model Context Protocol][3])

```yaml
apiVersion: joch.dev/v1alpha1
kind: Tool
metadata:
  name: github.create_issue
spec:
  type: mcp
  mcpServerRef:
    name: github

  description: Create a GitHub issue.

  inputSchema:
    type: object
    required:
      - repo
      - title
      - body
    properties:
      repo:
        type: string
      title:
        type: string
      body:
        type: string

  outputSchema:
    type: object
    properties:
      issueUrl:
        type: string
      issueNumber:
        type: integer

  sideEffects:
    level: external_write
    idempotent: false
    requiresApproval: true

  safety:
    allowedAgents:
      - engineering-agent
      - triage-agent
    deniedNamespaces:
      - sandbox
    rateLimit:
      callsPerMinute: 10

  observability:
    logArgs: true
    logResult: true
    redactFields:
      - token
      - password

status:
  phase: Ready
```

I would explicitly classify tools:

```text
read_only
local_write
external_write
financial
communication
code_execution
privileged
```

This matters for governance and approvals.

---

# 8. MCPServer spec

MCP servers expose tools, resources, and prompts to clients. Official MCP docs describe servers as offering features like resources, prompts, and tools. ([Model Context Protocol][1])

```yaml
apiVersion: joch.dev/v1alpha1
kind: MCPServer
metadata:
  name: github
spec:
  transport: streamable_http

  endpoint:
    url: https://mcp.example.com/github

  auth:
    type: oauth2
    secretRef:
      name: github-mcp-oauth

  exposes:
    tools: true
    resources: true
    prompts: false

  discovery:
    enabled: true
    refreshInterval: 10m

  security:
    sandbox: true
    allowStdio: false
    commandWhitelist: []
    pinServerVersion: true
    trustedPublisher: github

  policies:
    - name: mcp-tool-safety-policy

status:
  phase: Ready
  discoveredTools:
    - github.search_issues
    - github.create_issue
  discoveredResources:
    - github://repos
  lastDiscoveryAt: "2026-05-09T10:00:00Z"
```

I would be strict with MCP security. Recent reporting has highlighted MCP risks around local process execution, prompt injection, supply-chain poisoning, and unsafe STDIO handling, so the spec should make transport, trust, sandboxing, and command restrictions explicit. ([Tom's Hardware][4])

---

# 9. Memory spec

Memory should be split into different types.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Memory
metadata:
  name: research-agent-working-memory
spec:
  type: working

  scope:
    ownerKind: Agent
    ownerName: research-agent
    namespace: default

  backend:
    type: postgres
    connectionSecretRef:
      name: memory-postgres

  retention:
    ttl: 7d
    maxItems: 10000

  schema:
    fields:
      - name: content
        type: text
      - name: importance
        type: float
      - name: source
        type: string
      - name: expiresAt
        type: timestamp

  access:
    readableBy:
      - research-agent
      - reviewer-agent
    writableBy:
      - research-agent

  privacy:
    piiHandling: redact
    encryption: required

status:
  phase: Ready
  itemCount: 532
```

Memory types I would support:

```text
working      short-lived task context
episodic     history of past runs/events
semantic     durable facts and concepts
procedural   learned workflows/playbooks
preference   user/team preferences
blackboard   shared multi-agent workspace
```

Do not make all memory vector-based. Some memory should be relational, some document-based, some key-value, and some vector-searchable.

---

# 10. RAG spec

`RAG` should define retrieval behavior, not just vector storage.

```yaml
apiVersion: joch.dev/v1alpha1
kind: RAG
metadata:
  name: company-docs-rag
spec:
  sources:
    - name: engineering-docs
      kind: KnowledgeSource
    - name: product-docs
      kind: KnowledgeSource

  embeddingModelRef:
    name: text-embedding-large

  vectorStore:
    type: pgvector
    connectionSecretRef:
      name: rag-postgres
    collection: company_docs

  chunking:
    strategy: semantic
    maxTokens: 800
    overlapTokens: 120

  retrieval:
    topK: 12
    minScore: 0.72
    hybridSearch: true
    reranker:
      enabled: true
      modelRef:
        name: reranker-v1

  citations:
    required: true
    includeSourceUri: true
    includeLineNumbers: true

  freshness:
    syncInterval: 1h
    staleAfter: 7d

  permissions:
    inheritFromSource: true
    enforceAtQueryTime: true

status:
  phase: Ready
  documentsIndexed: 18342
  lastIndexedAt: "2026-05-09T09:45:00Z"
```

Separate `RAG` from `Memory`:

```text
Memory = agent/user/team state
RAG = retrieval over external knowledge corpora
```

---

# 11. KnowledgeSource spec

```yaml
apiVersion: joch.dev/v1alpha1
kind: KnowledgeSource
metadata:
  name: product-docs
spec:
  type: google_drive

  connector:
    secretRef:
      name: google-drive-connector

  include:
    paths:
      - /Product
      - /Engineering/RFCs

  exclude:
    patterns:
      - "**/Archive/**"
      - "**/*.tmp"

  sync:
    mode: incremental
    interval: 30m

  permissions:
    mode: source_inherited

  transforms:
    - extract_text
    - preserve_headings
    - detect_tables

status:
  phase: Ready
  lastSyncAt: "2026-05-09T09:45:00Z"
  documentsFound: 1220
```

This lets `joch` manage RAG pipelines cleanly.

---

# 12. Plan spec

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

# 13. ExecutionLoop spec

This is one we specifically mentioned, and it deserves its own resource.

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

# 14. Execution spec

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

# 15. Deployment spec

A `Deployment` runs agents as a managed fleet.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Deployment
metadata:
  name: research-agent-prod
spec:
  agentRef:
    name: research-agent

  replicas: 3

  strategy:
    type: rolling
    maxUnavailable: 1
    maxSurge: 1

  runtime:
    type: container
    image: ghcr.io/acme/agent-runtime:1.4.0

  environmentRef:
    name: prod-eu

  autoscaling:
    enabled: true
    minReplicas: 2
    maxReplicas: 20
    metrics:
      - type: queue_depth
        target: 20
      - type: latency_p95
        targetMs: 5000

  scheduling:
    region: eu
    nodeSelector:
      workload: agent-runtime

  rollout:
    canary:
      enabled: true
      percentage: 10
      duration: 30m

  healthChecks:
    readiness:
      type: synthetic_prompt
      prompt: "Reply with READY in JSON."
    liveness:
      type: heartbeat
      interval: 30s

status:
  phase: Available
  readyReplicas: 3
  currentRevision: 12
```

This is the resource that makes `joch` a real fleet manager, not just a local CLI.

---

# 16. Policy spec

Policies are critical for enterprise adoption.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Policy
metadata:
  name: research-agent-policy
spec:
  appliesTo:
    agents:
      - research-agent

  modelPolicy:
    allowedProviders:
      - openai
      - anthropic
    deniedModels:
      - experimental-unsafe-model

  dataPolicy:
    allowExternalDataTransfer: false
    pii:
      action: redact
    retention:
      maxDays: 30

  toolPolicy:
    default: deny
    allow:
      - web.search
      - browser.open
      - filesystem.write
    requireApproval:
      - github.create_issue
      - email.send
      - shell.exec

  networkPolicy:
    allowedDomains:
      - wikipedia.org
      - sec.gov
      - github.com

  budgetPolicy:
    maxCostUsdPerRun: 5
    maxCostUsdPerDay: 50

  audit:
    logPrompts: true
    logToolCalls: true
    logOutputs: true
    redactSecrets: true
```

Later, this could compile to OPA/Rego or Cedar-style policies.

---

# 17. Guardrail spec

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

# 18. Trace spec

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

we may also want `Event` as a lower-level append-only primitive.

---

# 19. Artifact spec

Agents produce durable things.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Artifact
metadata:
  name: market-analysis-report
spec:
  type: document
  producedBy:
    executionRef:
      name: exec-20260509-001

  storage:
    type: s3
    uri: s3://agent-artifacts/reports/market-analysis.md

  content:
    mimeType: text/markdown
    checksum: sha256:abc123

  provenance:
    sources:
      - https://example.com/source-1
    toolCalls:
      - call_123
      - call_456

status:
  phase: Ready
```

Artifacts give we reproducibility and auditability.

---

# 20. Secret spec

we can either wrap Kubernetes secrets or define our own reference object.

```yaml
apiVersion: joch.dev/v1alpha1
kind: SecretRef
metadata:
  name: openai-api-key
spec:
  provider: vault
  path: secret/data/openai
  key: apiKey
```

I would avoid storing actual secrets in `joch` specs.

---

# 21. Budget spec

Cost control should be a first-class resource.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Budget
metadata:
  name: research-team-budget
spec:
  scope:
    namespace: research

  limits:
    dailyUsd: 100
    monthlyUsd: 2000
    perExecutionUsd: 10
    tokenLimitDaily: 10000000

  actions:
    onSoftLimit: warn
    onHardLimit: suspend

  allocation:
    byAgent:
      research-agent: 0.5
      writer-agent: 0.3
      reviewer-agent: 0.2

status:
  spentTodayUsd: 42.12
  spentMonthUsd: 880.44
```

---

# 22. Eval spec

we need evals to prevent silent regressions.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Eval
metadata:
  name: research-agent-citation-eval
spec:
  target:
    kind: Agent
    name: research-agent

  dataset:
    type: file
    uri: ./evals/research-cases.jsonl

  metrics:
    - name: factuality
      type: llm_judge
    - name: citation_accuracy
      type: deterministic
    - name: task_success
      type: llm_judge
    - name: cost
      type: numeric
    - name: latency
      type: numeric

  schedule:
    onDeployment: true
    nightly: true

  thresholds:
    factuality: 0.9
    citation_accuracy: 0.95
    task_success: 0.85

status:
  phase: Passed
  lastRunAt: "2026-05-09T02:00:00Z"
  scores:
    factuality: 0.93
    citation_accuracy: 0.97
```

---

# 23. Environment spec

```yaml
apiVersion: joch.dev/v1alpha1
kind: Environment
metadata:
  name: prod-eu
spec:
  region: eu-central

  runtime:
    orchestrator: kubernetes
    cluster: prod-agent-cluster

  defaults:
    modelRef:
      name: gpt-5-thinking
    traceRetentionDays: 30
    logLevel: info

  compliance:
    dataResidency: EU
    piiMode: redact
    auditRequired: true
```

This lets we do:

```bash
joch deploy research-agent --env prod-eu
```

---

# 24. Team / Namespace spec

Eventually we will need ownership.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Team
metadata:
  name: growth
spec:
  members:
    - user: alice@example.com
      role: admin
    - user: bob@example.com
      role: operator

  namespaces:
    - growth-dev
    - growth-prod

  defaultPolicies:
    - budget-policy
    - pii-policy
```

---

# 25. Resource relationships

The graph should look like this:

```text
Deployment
  └── Agent
        ├── Model
        ├── Personality
        ├── Prompt
        ├── Skills
        │     └── Tools
        │           └── MCPServer
        ├── Memories
        ├── RAG
        │     └── KnowledgeSources
        ├── ExecutionLoop
        ├── Policies
        ├── Guardrails
        └── Budgets

Execution
  ├── Agent
  ├── Plan
  ├── Trace
  ├── ToolCalls
  ├── MemoryWrites
  └── Artifacts
```

This separation is important because different resources change at different speeds:

```text
Model: changes often
Personality: changes carefully
Policy: changes through governance
Memory: changes continuously
Execution: created per run
Deployment: changes through rollout
```

---

# 26. Add `ToolCall` as a runtime resource

This is easy to miss. Tool calls deserve durable records.

```yaml
apiVersion: joch.dev/v1alpha1
kind: ToolCall
metadata:
  name: call-abc123
spec:
  executionRef:
    name: exec-20260509-001

  agentRef:
    name: research-agent

  toolRef:
    name: github.create_issue

  input:
    repo: acme/joch
    title: Add Memory spec
    body: Define working, semantic, and episodic memory resources.

  sideEffects:
    level: external_write
    idempotencyKey: issue-joch-memory-spec-001

  approval:
    required: true
    status: approved
    approvedBy: alice@example.com

status:
  phase: Succeeded
  startedAt: "2026-05-09T10:35:00Z"
  completedAt: "2026-05-09T10:35:03Z"
  output:
    issueUrl: https://github.com/acme/joch/issues/123
```

Why this matters:

```text
No duplicate side effects
Auditability
Replay safety
Cost attribution
Security review
Migration across vendors
```

---

# 27. Add `Handoff` as a runtime resource

Multi-agent systems need explicit handoff records.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Handoff
metadata:
  name: handoff-001
spec:
  executionRef:
    name: exec-20260509-001

  fromAgent:
    name: research-agent

  toAgent:
    name: writer-agent

  reason: >
    Research phase complete; writer-agent should draft final report.

  context:
    summary: >
      Found 12 competing tools and 4 differentiation themes.
    artifactRefs:
      - market-research-notes
    memoryRefs:
      - research-agent-working-memory

  expectedOutput:
    type: markdown_report

status:
  phase: Accepted
```

---

# 28. Add `Approval` as a resource

Human-in-the-loop should not be bolted on.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Approval
metadata:
  name: approval-send-email-001
spec:
  requestedBy:
    executionRef:
      name: exec-20260509-001

  action:
    type: tool_call
    toolCallRef:
      name: call-send-email-001

  risk:
    level: external_write
    summary: Agent wants to send an email to a customer.

  options:
    - approve
    - reject
    - edit

  expiresAfter: 2h

status:
  phase: Pending
```

Then:

```bash
joch approvals ls
joch approvals approve approval-send-email-001
```

---

# 29. Add `Conversation` as a resource

For vendor-agnostic state persistence, this is essential.

```yaml
apiVersion: joch.dev/v1alpha1
kind: Conversation
metadata:
  name: conv-123
spec:
  agentRef:
    name: research-agent

  stateStore:
    type: postgres
    ref: conversation-store-prod

  retention:
    maxDays: 90

  summarization:
    enabled: true
    checkpointEveryMessages: 20
    modelRef:
      name: gpt-5-mini

  portability:
    canonicalFormat: joch.dev/conversation.v1
    preserveVendorMetadata: true

status:
  phase: Active
  messageCount: 82
  currentBackend: openai:gpt-5-thinking
  lastCheckpoint: chk-456
```

---

# 30. Add `StateCheckpoint`

This supports model/provider switching.

```yaml
apiVersion: joch.dev/v1alpha1
kind: StateCheckpoint
metadata:
  name: chk-456
spec:
  conversationRef:
    name: conv-123

  executionRef:
    name: exec-20260509-001

  summary:
    userGoal: Design joch resource specs.
    currentTask: Draft Kubernetes-style YAML specifications.
    decisions:
      - Use vendor-neutral state.
      - Treat tools, skills, and MCP servers separately.
      - Persist tool calls as runtime resources.
    openQuestions:
      - Which resources belong in v1alpha1?

  activeArtifacts:
    - joch-resource-spec-draft

  memoryRefs:
    - research-agent-working-memory

  providerState:
    from:
      provider: openai
      model: gpt-5-thinking
    portable: true

status:
  phase: Ready
```

---

# 31. Suggested v1alpha1 API groups

I would organize the API into groups:

```text
joch.dev/v1alpha1
  Agent
  Personality
  Prompt
  Skill

runtime.joch.dev/v1alpha1
  Execution
  Plan
  ExecutionLoop
  ToolCall
  Handoff
  Approval
  Conversation
  StateCheckpoint

model.joch.dev/v1alpha1
  Model
  ModelRoute
  ModelPolicy

tools.joch.dev/v1alpha1
  Tool
  MCPServer
  ToolRegistry

memory.joch.dev/v1alpha1
  Memory
  RAG
  KnowledgeSource

ops.joch.dev/v1alpha1
  Deployment
  Environment
  Budget
  Trace
  Eval
  Artifact
  Policy
  Guardrail
```

For a CLI, the user would still see simple commands:

```bash
joch get agents
joch get models
joch get skills
joch get memories
joch get rags
joch get executions
joch get deployments
```

---

# 32. What we missed

we already listed most of the obvious resources. I would add these:

```text
Prompt
Conversation
StateCheckpoint
Policy
Guardrail
Budget
Trace
Artifact
Approval
Handoff
KnowledgeSource
Eval
Environment
SecretRef
ModelRoute
ToolCall
```

The most important missed ones are:

```text
Conversation
StateCheckpoint
Policy
Guardrail
Trace
Approval
ToolCall
Artifact
```

Without those, we cannot get serious enterprise governance, reproducibility, or vendor migration.

---

# 33. Example complete app bundle

A real project might look like:

```text
joch.yaml
agents/
  research-agent.yaml
  writer-agent.yaml
models/
  openai-gpt.yaml
  anthropic-claude.yaml
personalities/
  pragmatic-researcher.yaml
  technical-writer.yaml
prompts/
  research-system.yaml
skills/
  web-research.yaml
  report-writing.yaml
tools/
  web-search.yaml
  github-tools.yaml
mcp/
  github.yaml
  slack.yaml
memory/
  research-memory.yaml
rag/
  company-docs-rag.yaml
policies/
  default-policy.yaml
  pii-policy.yaml
guardrails/
  citation-required.yaml
deployments/
  research-agent-prod.yaml
evals/
  research-agent-eval.yaml
```

Then:

```bash
joch apply -f .
joch get agents
joch run research-agent -f tasks/market-research.yaml
joch logs execution/exec-123
joch trace execution/exec-123
joch approvals ls
```

---

# 34. The north-star spec model

The cleanest mental model is:

```text
Desired state:
  Agent, Model, Skill, Personality, Memory, RAG, Policy, Deployment

Runtime state:
  Execution, Plan, ToolCall, Handoff, Approval, Trace, Artifact

Portable state:
  Conversation, StateCheckpoint, Memory, Artifact

Integration state:
  Tool, MCPServer, KnowledgeSource, SecretRef
```

That gives `joch` a real control-plane architecture.

The core design philosophy should be:

> Agents are not prompts.
> Agents are deployable, observable, governed, stateful resources.

That is the difference between a CLI wrapper and a true agent fleet manager.

[1]: https://modelcontextprotocol.io/specification/2025-11-25?utm_source=chatgpt.com "Specification"
[2]: https://openai.github.io/openai-agents-python/tracing/?utm_source=chatgpt.com "Tracing - OpenAI Agents SDK"
[3]: https://modelcontextprotocol.io/specification/2025-11-25/server/tools?utm_source=chatgpt.com "Tools"
[4]: https://www.tomshardware.com/tech-industry/artificial-intelligence/anthropics-model-context-protocol-has-critical-security-flaw-exposed?utm_source=chatgpt.com "Anthropic's Model Context Protocol includes a critical remote code execution vulnerability - newly discovered exploit puts 200,000 AI servers at risk"
