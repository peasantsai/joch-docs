# FAQ

## Positioning

### Is Joch another agent SDK?

No. Joch is the **control plane** that operates agents built with OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, CrewAI, or custom code. Agent code stays in the SDK. See [Positioning](../concepts/positioning.md).

### Should I rewrite my agent in Joch?

No. Onboard with one line per agent — `bind_runtime(...)`, `joch_options(...)`, `.UseJoch(...)` — and keep the rest of the SDK code identical. See [Onboard from a Vendor SDK](../getting-started/migration-from-vendor-sdk.md).

### Does Joch host inference?

No. Joch routes calls to providers (OpenAI, Anthropic, Google, Microsoft Foundry, Ollama, vLLM, llama.cpp). Inference billing stays with the provider. See [Providers](../providers/index.md).

## OWASP AOS

### What is OWASP AOS?

The Agent Observability Standard. Three pillars: **Inspect** (AgBOM), **Instrument** (hooks with `allow` / `deny` / `modify`), **Trace** (events extending OpenTelemetry and OCSF). Joch implements the standard. See [AOS Conformance](../aos/index.md).

### Is it AgBOM or ABOM?

**AgBOM**. The OWASP standard uses AgBOM (Agent Bill of Materials). Joch's resource kind is named `AgBOM`. See [AgBOM](../aos/agbom.md) and [`AgBOM` resource](../specs/kubernetes/agbom.md).

### Can I plug in my own Guardian Agent?

Yes. The default Guardian is Joch's policy engine. Third-party Guardians can substitute or compose: any `deny` is a `deny`; `modify` outputs are merged in declared order. See [Hooks](../aos/hooks.md).

## Resources and architecture

### Why are there so many resource kinds?

Each kind models one operational concern (identity, tool, MCP server, model route, policy, approval, AgBOM, trace, etc.). Operations is the value proposition; one big "Agent" record would erase the audit trail.

### Why is `Personality` / `Skill` / `Prompt` / `Plan` not a resource kind?

Those belong inside the SDK. Modeling them as control-plane resources would force every SDK into one shape and pull Joch into the SDK competition. See [Resource Model](../specs/resource-model.md).

### Why YAML?

Kubernetes-style YAML is the lingua franca of platform engineering, and the resources work the same against SQLite, Postgres, and Kubernetes CRDs.

## Operations

### Where do secrets live?

Never in Joch. `Secret` is a reference to Vault / Kubernetes Secret / AWS Secrets Manager / GCP Secret Manager / Azure Key Vault / env. Resolution happens at the data-plane gateway boundary. See [Trust and Security Model](../architecture/trust-and-security-model.md).

### Where do conversations live?

In `Conversation` records inside Joch. The SDK's session storage continues to work; Joch additionally writes a portable canonical event log. See [State Portability](../architecture/state-portability.md).

### Can I switch providers mid-conversation?

Yes. Joch creates a deterministic `StateCheckpoint`, validates target capabilities, and resumes on the new provider without replaying side-effecting tool calls. See [Cross-Provider Migration](../use-cases/cross-provider-migration.md).

### How do I prove a tool call did not happen?

The audit log + trace + ToolCall record together. Replay any execution from inputs to outputs, with policy version, hook decisions, and approval evidence. See [Observability](../pillars/observability.md).

## Performance and scale

### Latency overhead?

The policy engine is sub-millisecond p95. The biggest tax is one extra hop per cross-boundary call (tool, MCP, model, memory, RAG). In practice this is negligible compared to model latency.

### How many agents can a single Joch deployment govern?

In single-binary mode, thousands. With horizontal scale-out of the data-plane services, tens of thousands. The control plane scales independently of the data plane.

## Compliance

### Is Joch SOC 2 / ISO 27001 / FedRAMP?

Joch Open Source is not certified per se; certifications attach to operational deployments. Joch Cloud's certification roadmap: SOC 2 Type 1 → SOC 2 Type 2 → ISO 27001 → FedRAMP Moderate. See [Roadmap](../business/roadmap.md).

### Does Joch help with the EU AI Act / AI BoM requirements?

AgBOM, audit trail, and OCSF-aligned security events are designed to make compliance evidence collection straightforward. The exact mapping to specific regulations is the customer's responsibility, but the primitives are aligned with the standards regulators are converging on.

## Open source and licensing

### What's the license?

Apache-2.0 across all `peasantsai/joch*` open-source repositories. `peasantsai/joch-cloud` is the commercial repository. See [License](license.md).

### Can I self-host?

Yes. The OSS core is sufficient for production self-hosted deployments, including with a Kubernetes operator. See [Deployment Modes](../platform/deployment-modes.md).

### Will you relicense to BSL / SSPL?

No. We chose Apache-2.0 deliberately and intend to stay there. Trademark protection is the durable lever, not relicensing. See [Open-Core Strategy](../business/open-core.md).

## Help

### Where do I file a bug?

Per repository: `peasantsai/joch`, `peasantsai/joch-spec`, `peasantsai/joch-docs`, `peasantsai/joch-examples`, `peasantsai/joch-registry`. See [Contributing](contributing.md).

### Where do I get help?

GitHub Discussions, the Joch Slack / Discord (linked in repository READMEs), and monthly maintainer office hours.
