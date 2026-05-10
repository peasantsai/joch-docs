# Environment Variables

All Joch environment variables use the `JOCH_` prefix. Provider keys use the provider's own conventions.

## CLI

| Variable | Default | Purpose |
|---|---|---|
| `JOCH_CONTEXT` | active context | Override the active context for one command. |
| `JOCH_SERVER` | from context | Override `joch-server` URL. |
| `JOCH_NAMESPACE` | from context | Default namespace. |
| `JOCH_OUTPUT` | `table` | Default output format. |
| `JOCH_NO_COLOR` | unset | Disable colored output. |
| `JOCH_DEBUG` | unset | Verbose debug logs. |
| `JOCH_OIDC_ID_TOKEN` | unset | Static OIDC token (CI use). |

## `joch-server`

| Variable | Default | Purpose |
|---|---|---|
| `JOCH_LISTEN_PORT` | `8080` | API listen port. |
| `JOCH_TLS_CERT` | unset | TLS cert path. |
| `JOCH_TLS_KEY` | unset | TLS key path. |
| `JOCH_OIDC_ISSUER` | unset | OIDC issuer URL. |
| `JOCH_OIDC_AUDIENCE` | unset | OIDC audience. |
| `JOCH_STORE` | `sqlite` | `sqlite` / `postgres` / `kubernetes-crd`. |
| `JOCH_DATABASE_URL` | unset | Postgres DSN. |
| `JOCH_EVENT_BUS` | `postgres` | `postgres` / `redis` / `nats` / `kafka`. |
| `JOCH_AUDIT_RETENTION_DAYS` | `365` | Audit retention. |

## `joch-worker`

| Variable | Default | Purpose |
|---|---|---|
| `JOCH_SERVER_URL` | required | Control plane URL. |
| `JOCH_RUNTIME_ADAPTER` | `local` | `local` / `docker` / `kubernetes` / `nomad` / `serverless`. |
| `JOCH_WORKER_POOL` | `default` | Pool name. |
| `JOCH_WORKER_CONCURRENCY` | `8` | Concurrent executions. |

## `joch-gateway`

| Variable | Default | Purpose |
|---|---|---|
| `JOCH_GATEWAY_PORT` | `8081` | Listen port. |
| `JOCH_TOOL_RATE_BURST` | `50` | Token-bucket burst. |
| `JOCH_TOOL_RATE_PER_SECOND` | `25` | Token-bucket rate. |
| `JOCH_MCP_PIN_BY_DEFAULT` | `true` | Require pinned MCP versions. |
| `JOCH_MCP_INJECTION_SCAN` | `true` | Run prompt-injection scan on inbound results. |

## `joch-router`

| Variable | Default | Purpose |
|---|---|---|
| `JOCH_ROUTER_PORT` | `8083` | Listen port. |
| `JOCH_PROVIDER_OPENAI_BASE_URL` | upstream default | OpenAI base URL override. |
| `JOCH_PROVIDER_ANTHROPIC_BASE_URL` | upstream default | Anthropic base URL. |
| `JOCH_PROVIDER_GOOGLE_BASE_URL` | upstream default | Google base URL. |
| `JOCH_PROVIDER_MICROSOFT_FOUNDRY_ENDPOINT` | unset | Microsoft Foundry project endpoint. |
| `JOCH_PROVIDER_OLLAMA_BASE_URL` | `http://localhost:11434` | Ollama URL. |

## `joch-memory`

| Variable | Default | Purpose |
|---|---|---|
| `JOCH_MEMORY_BACKEND` | `postgres` | `postgres` / `pgvector` / `redis` / `qdrant`. |
| `JOCH_VECTOR_BACKEND` | `pgvector` | Vector store. |

## `joch-trace`

| Variable | Default | Purpose |
|---|---|---|
| `JOCH_OTLP_GRPC_PORT` | `4317` | OTLP gRPC ingest. |
| `JOCH_OTLP_HTTP_PORT` | `4318` | OTLP HTTP ingest. |
| `JOCH_TRACE_RETENTION_DAYS` | `30` | Trace retention. |

## Provider keys

| Variable | Provider |
|---|---|
| `OPENAI_API_KEY` | OpenAI |
| `ANTHROPIC_API_KEY` | Anthropic |
| `GOOGLE_APPLICATION_CREDENTIALS` | Google service account |
| `GOOGLE_CLOUD_PROJECT` | Google |
| `AZURE_FOUNDRY_PROJECT_ENDPOINT` | Microsoft Foundry |
| `AZURE_TENANT_ID` / `AZURE_CLIENT_ID` / `AZURE_CLIENT_SECRET` | Azure auth |
| `AWS_ACCESS_KEY_ID` / `AWS_SECRET_ACCESS_KEY` / `AWS_REGION` | AWS Bedrock |
| `OLLAMA_HOST` | Ollama (`http://localhost:11434`) |

These are read at the data-plane gateway boundary and never logged. To rotate, update the underlying secret store; Joch does not cache the value.

## Diagnostics

| Variable | Default | Purpose |
|---|---|---|
| `JOCH_PROFILE` | unset | Enable pprof endpoint at `/debug/pprof`. |
| `JOCH_LOG_LEVEL` | `info` | `debug` / `info` / `warn` / `error`. |
| `JOCH_LOG_FORMAT` | `json` | `json` / `text`. |
| `JOCH_TRACE_SDK` | unset | Send Joch's own internal traces to a collector. |
