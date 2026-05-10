# Install

Joch supports four install paths. Pick the one that matches where you want to run agents — same `joch apply -f .` works against all four.

## Local (developer mode)

Single process, SQLite store, local files, no Kubernetes.

```bash
brew install joch                       # macOS / Homebrew
# or
curl -sSL https://joch.dev/install.sh | bash
```

Start the local control plane and apply a sample agent:

```bash
joch up
joch apply -f https://raw.githubusercontent.com/peasantsai/joch-examples/main/quickstart/support-triage.yaml
joch run support-triage "Triage incoming queue"
```

## Docker Compose

For team dev environments, CI smoke tests, and small self-hosted deployments.

```bash
joch init docker-compose
docker compose up -d
joch context use docker
joch apply -f .
```

Stack: `joch-server`, `joch-worker`, `joch-gateway`, `joch-router`, `joch-memory`, `joch-trace`, Postgres (with pgvector), Redis (event bus), MinIO (artifacts), OpenTelemetry collector.

## Kubernetes

For production. Native CRDs, Helm chart, controllers, NetworkPolicies, ServiceMonitor, autoscaling.

```bash
helm repo add joch https://charts.joch.dev
helm install joch joch/joch -n joch-system --create-namespace
joch context use kubernetes
joch apply -f .
```

## Joch Cloud (managed control plane)

Hosted multi-tenant SaaS plus customer-runtime tunnel. Free Starter tier, paid Team / Enterprise tiers.

```bash
joch cloud login
joch runtime install kubernetes --name prod-eu
joch apply -f . --context cloud
```

See [Revenue Models](../business/revenue-models.md) for tier details.

## Verify the install

```bash
joch doctor
```

The doctor checks:

- API server reachable,
- model credentials configured,
- registered MCP servers reachable and healthy,
- worker registered and ready,
- trace endpoint reachable,
- AgBOM service ready.

## Configure model providers

Joch resolves provider credentials at the gateway boundary. Set one of:

```bash
# OpenAI
export OPENAI_API_KEY=sk-...

# Anthropic
export ANTHROPIC_API_KEY=sk-ant-...

# Google
export GOOGLE_APPLICATION_CREDENTIALS=/path/to/key.json
export GOOGLE_CLOUD_PROJECT=...

# Microsoft Foundry
az login
export AZURE_FOUNDRY_PROJECT_ENDPOINT=https://...
```

Or apply [`Secret`](../specs/kubernetes/secret.md) records pointing at Vault, Kubernetes secrets, or AWS / GCP / Azure secret managers. Joch never stores secret values.

## Next

- [Build your first agent](first-agent.md)
- [Govern an MCP server](governing-mcp.md)
- [Onboard from a vendor SDK](migration-from-vendor-sdk.md)
