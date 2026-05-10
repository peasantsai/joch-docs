# First Agent

This walkthrough takes about 15 minutes. You will register a Joch-governed agent that calls one tool, with a portable `Policy`, a `ModelRoute`, and a generated `AgBOM`.

## 1. Apply foundation resources

Save as `bootstrap.yaml`:

```yaml
apiVersion: joch.dev/v1alpha1
kind: FrameworkAdapter
metadata: { name: openai-agents-sdk }
spec:
  framework: openai-agents-sdk
  language: python
  adapterImage: ghcr.io/peasantsai/joch-adapter-openai:1.0.0
  capabilities:
    toolBinding: true
    mcpBinding: true
    modelRouterBinding: true
    streaming: true
---
apiVersion: model.joch.dev/v1alpha1
kind: Model
metadata: { name: gpt-5-thinking }
spec:
  provider: openai
  model: gpt-5-thinking
  auth:
    secretRef: { name: openai-api-key }
  capabilities:
    text: true
    toolCalling: true
    structuredOutput: true
    streaming: true
  limits:
    contextWindowTokens: 400000
---
apiVersion: model.joch.dev/v1alpha1
kind: ModelRoute
metadata: { name: research-default }
spec:
  requirements: { toolCalling: true, contextWindowMin: 200000 }
  strategy:
    primary: openai:gpt-5-thinking
  constraints:
    maxCostUsdPerExecution: 5
---
apiVersion: ops.joch.dev/v1alpha1
kind: Secret
metadata: { name: openai-api-key }
spec:
  source:
    type: env
    name: OPENAI_API_KEY
```

Apply:

```bash
joch apply -f bootstrap.yaml
```

## 2. Define a tool

Save as `tool.yaml`:

```yaml
apiVersion: tools.joch.dev/v1alpha1
kind: Tool
metadata: { name: web.search }
spec:
  type: rest
  endpoint:
    method: GET
    url: https://api.example.com/search
    auth: { type: bearer, secretRef: { name: web-search-key } }
  inputSchema:
    type: object
    required: [q]
    properties:
      q: { type: string }
  outputSchema:
    type: object
    properties:
      results:
        type: array
        items:
          type: object
          properties:
            url:   { type: string }
            title: { type: string }
            snippet: { type: string }
  sideEffects: { level: read_only, idempotent: true }
---
apiVersion: ops.joch.dev/v1alpha1
kind: Secret
metadata: { name: web-search-key }
spec:
  source: { type: env, name: WEB_SEARCH_KEY }
```

```bash
joch apply -f tool.yaml
```

## 3. Define a policy

Save as `policy.yaml`:

```yaml
apiVersion: joch.dev/v1alpha1
kind: Policy
metadata: { name: research-cost-cap }
spec:
  appliesTo:
    agents: { selector: { matchLabels: { team: research } } }
  rules:
    - when: { cost.runUsd: ">3" }
      action: { deny: true, reason: research-cap-exceeded }
    - when: { tool.sideEffect: external_write }
      require: { approval: human }
```

```bash
joch apply -f policy.yaml
```

## 4. Register the agent

Save as `agent.yaml`:

```yaml
apiVersion: joch.dev/v1alpha1
kind: Agent
metadata:
  name: market-researcher
  labels: { team: research, env: dev }
spec:
  description: Researches markets using web.search and writes a structured summary.
  framework:
    adapterRef: { name: openai-agents-sdk }
    pythonModule: my_agents.researcher:agent
    image: ghcr.io/example/market-researcher:0.1.0
  modelRoute:
    routeRef: { name: research-default }
  tools:
    - name: web.search
  policies:
    - name: research-cost-cap
  observability:
    tracing: enabled
    agbom: enabled
```

```bash
joch apply -f agent.yaml
```

## 5. Run it

```bash
joch run market-researcher "Summarize the agent operations control-plane market in 10 bullet points."
```

While it runs:

```bash
joch trace last
joch get toolcalls --agent market-researcher --since 5m
```

After it completes:

```bash
joch agbom market-researcher
joch agbom market-researcher --format cyclonedx > market-researcher.cdx.json
joch cost by-agent --since 1h
```

## 6. Try a denial

Add a rule to the policy that blocks `web.search`:

```yaml
    - when: { tool.name: "web.search" }
      action: { deny: true, reason: web-search-disabled-temporarily }
```

```bash
joch apply -f policy.yaml
joch run market-researcher "Try again."
joch denials ls --policy research-cost-cap --since 5m
```

The denial appears in the trace with policy id and version. Restore the policy when you are done.

## What just happened

- `Agent`, `Policy`, `ModelRoute`, `Tool`, and `Secret` records are versioned in the Joch resource store.
- Every `web.search` call went through the [Tool Gateway](../architecture/tool-gateway.md) and emitted AOS `steps/toolCallRequest` and `steps/toolCallResult` hooks.
- Every model call went through the [Model Router](../architecture/model-router.md) and was capped by the policy's cost rule.
- `AgBOM` was generated with the model, the tool, the framework adapter version, and the policy version pinned at run time.

## Next

- [Govern an MCP server](governing-mcp.md)
- [Onboard from a vendor SDK](migration-from-vendor-sdk.md)
- [SDK Integrations](../integrations/index.md)
