# Troubleshooting

A focused list of failure modes and how to diagnose them. Run `joch doctor` first — it catches most setup issues.

## Install and connectivity

### `joch up` fails immediately

```text
Error: bind: address already in use
```

Another process holds port `8080`. Stop it or set `JOCH_LISTEN_PORT`.

### CLI cannot reach the server

```text
Error: dial tcp 127.0.0.1:8080: connect: connection refused
```

```bash
joch context current
joch context set local --server http://localhost:8080
```

### Helm install in Kubernetes hangs

CRD installation is the most common stall point. Re-run with `--wait --timeout 5m` and inspect:

```bash
kubectl get crd | grep joch.dev
kubectl describe deployment -n joch-system joch-server
kubectl logs -n joch-system deploy/joch-controller-manager
```

## Apply and validate

### `joch apply` rejects with "missing referenced resource"

Apply foundation records first: `FrameworkAdapter`, `Model`, `ModelRoute`, `Secret`, `Tool`, `MCPServer`. Then `Agent`, `Policy`, `Deployment`.

### `joch validate` says "selector matches no agents"

`Policy.appliesTo.agents.selector` does not match any labels currently on `Agent` records. Re-check labels with `joch get agents --show-labels`.

### `joch promote` fails on eval gate

```text
PromotionFailed: eval support-triage-quality below threshold task_success=0.82 < 0.85
```

Investigate the failing eval before re-running. Promotion gates are non-bypassable except by an explicit `--force` (audited) or by adjusting the threshold.

## MCP gateway

### Agent reports tool not found

```text
MCPServerError: tool github.create_issue not found on server github
```

The pinned version may not expose that tool. Refresh discovery and check:

```bash
joch mcp discover github
joch describe mcpserver github
```

### Schema drift quarantines a server

```text
PolicyDenied: schema drift detected for github
```

Review the drift and decide:

```bash
joch mcp upgrade github --to <new-version> --review
joch mcp pin github --to <decided-version>
```

To restore service before review, lift quarantine with an audited override.

### MCP server times out

```text
MCPServerTimeout: github exceeded request timeout 30s
```

Check the server's health, network policy, and the `MCPServer.spec.endpoint.url`. The gateway does not retry automatically for non-idempotent tool calls.

## Tool gateway

### Tool call denies unexpectedly

Read the denial reason in the trace:

```bash
joch denials ls --since 5m
joch describe toolcall <id>
```

Common causes:

- side-effect class mismatch (`external_write` triggered approval),
- missing approval (timeout reached `onTimeout: deny`),
- policy update active in this `Environment`.

### Approval never resolves

```text
Approval status: Pending for 35m (timeout 30m)
```

The approval routing channel may be unreachable. Check:

```bash
joch describe approval <id>
joch describe notificationchannel <name>
```

Also confirm at least one approver responded; channel-only delivery does not auto-decide.

## Model router

### `ProviderSwitched` fires too aggressively

The `ModelRoute.spec.strategy.failover` triggers may be too broad. Tighten:

```yaml
failover:
  onError: true
  onBudgetExceeded: true
  onCapabilityMismatch: true
  onRegionMismatch: true
  onHealthDegraded: false   # was: true
```

### `BudgetExceeded` blocks every call

`Budget.spec.limits.perRunUsd` may be too tight. Inspect with:

```bash
joch cost by-agent --since 1h
joch describe budget <name>
```

### Capability mismatch

```text
CapabilityMismatch: ModelRoute requires structuredOutput, model anthropic:claude-sonnet declares structuredOutput=false
```

Either change the route's `requirements`, declare the capability on the `Model` record, or pick a different fallback.

## Memory and RAG

### Memory writes denied

A `Policy` may be redacting PII or outright blocking writes by class. Check denial events on `joch trace last`.

### RAG retrievals empty

```text
joch rag retrieve support-docs-rag --query "refund policy"
```

Confirm the `KnowledgeSource` last sync succeeded (`joch describe knowledgesource ...`) and the `RAG.status.documentCount > 0`.

## AgBOM

### AgBOM not refreshing

```bash
joch describe agent <name>
```

Check `status.agbomRef` is set. Force refresh:

```bash
curl -XPOST http://joch-server:8080/v1/agbom/<agent>/refresh
```

If the agent record has `observability.agbom: disabled`, re-enable it.

### CycloneDX consumer rejects the document

The `joch.*` properties are extensions and should be ignored by tools that do not understand them. If the consumer is strict-validating:

```bash
joch agbom <agent> --format cyclonedx --extensions=off > <agent>.cdx.json
```

## Trace and observability

### No spans show up in Honeycomb / Datadog / Grafana

Check the Trace record's exporter config:

```bash
joch describe trace <name>
joch logs deploy/joch-trace
```

Verify the OTLP endpoint, headers, and TLS. The Joch trace store still receives spans even when external export fails; cross-check via `joch trace <id>`.

### Cost looks wrong

Cost is computed from `Model.spec.pricing`. Update the pricing block when providers change prices; the router uses the values active at the time of each call.

## Provider issues

### `thinking.type.enabled` API error (Claude Agent SDK)

The Claude Agent SDK requires v0.2.111+ for Opus 4.7. Upgrade the SDK in the agent runtime image.

### Azure / Foundry 401

Re-run `az login`, ensure the service principal has `Cognitive Services User` (or equivalent) on the project, and confirm `AZURE_FOUNDRY_PROJECT_ENDPOINT` is set.

### Vertex region mismatch

```text
ProviderSwitched: region mismatch (Environment.dataResidency=EU; model region=us-central1)
```

Add an EU-region `Model` record and put it in the route's `fallback`, or switch `dataResidency`.

## When all else fails

Open a GitHub issue against the relevant repo with:

- `joch version`
- `joch context current`
- the exact command and full output,
- the resource YAMLs involved (with secrets redacted),
- the `joch trace <id>` output for the failing execution.

See [Contributing](contributing.md) for issue triage cadence.
