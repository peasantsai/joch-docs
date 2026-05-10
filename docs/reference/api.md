# API Reference

Joch exposes a REST + gRPC API surface, plus a Watch API for resource changes. Full OpenAPI and Proto definitions live in [`peasantsai/joch-spec`](https://github.com/peasantsai/joch-spec).

## Base URLs

```text
control plane (joch-server)        http(s)://<server>/v1
tool gateway (joch-gateway)         http://joch-gateway:8081
mcp gateway  (joch-gateway)         http://joch-gateway:8082
model router (joch-router)          http://joch-router:8083
memory       (joch-memory)          http://joch-memory:8084
rag          (joch-memory)          http://joch-memory:8085
trace        (joch-trace)           OTLP-HTTP/grpc; query at http://joch-trace:8086
```

All endpoints use TLS in production. Auth is OIDC by default; service-to-service uses workload identities.

## Resource API (control plane)

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/v1/resources/apply` | Create or update one or more resources (multi-doc YAML body). |
| `POST` | `/v1/resources/validate` | Validate without applying. |
| `POST` | `/v1/resources/diff` | Structural diff against current state. |
| `GET` | `/v1/namespaces/{ns}/{kind}` | List resources of a kind. |
| `GET` | `/v1/namespaces/{ns}/{kind}/{name}` | Get a resource. |
| `PATCH` | `/v1/namespaces/{ns}/{kind}/{name}` | Patch (`spec` only; `status` is a subresource). |
| `DELETE` | `/v1/namespaces/{ns}/{kind}/{name}` | Delete. |
| `WATCH` | `/v1/resources?selector=...` | Streaming change feed. |

Resource kinds use the API groups documented in [Platform → CRDs and Controllers](../platform/crds-controllers.md).

## Execution API

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/v1/namespaces/{ns}/executions` | Create + run an execution. |
| `GET` | `/v1/namespaces/{ns}/executions/{name}` | Get an execution. |
| `GET` | `/v1/namespaces/{ns}/executions` | List executions (filters: `agent`, `phase`, `since`). |
| `POST` | `/v1/namespaces/{ns}/executions/{name}/cancel` | Cancel. |
| `GET` | `/v1/namespaces/{ns}/executions/{name}/events` | Streamed event log. |
| `POST` | `/v1/namespaces/{ns}/executions/{name}/replay` | Replay (policy-gated). |

## Tool gateway API

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/v1/tool-calls` | Execute a tool through the gateway. |
| `GET` | `/v1/tool-calls/{id}` | Inspect a tool call. |
| `POST` | `/v1/tool-calls/{id}/cancel` | Cancel. |
| `POST` | `/v1/tool-calls/{id}/approve` | Resume after approval. |

## MCP gateway API

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/v1/mcp/discover` | Refresh discovered capabilities. |
| `POST` | `/v1/mcp/{server}/call` | Proxied JSON-RPC `tools/call`. |
| `POST` | `/v1/mcp/{server}/pin` | Pin a version. |
| `POST` | `/v1/mcp/{server}/quarantine` | Quarantine. |

## Model router API

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/v1/model/respond` | Non-streaming model call. |
| `POST` | `/v1/model/stream` | Streaming model call. |
| `GET` | `/v1/models/capabilities` | Capability vector for matching. |
| `GET` | `/v1/route/{name}/explain?execution={id}` | Explain a route decision. |

## Memory API

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/v1/memory/{name}/query` | Read memory items. |
| `POST` | `/v1/memory/{name}/write` | Write a memory item. |
| `DELETE` | `/v1/memory/{name}/items/{id}` | Delete an item. |
| `POST` | `/v1/memory/{name}/summarize` | Run a summary pass. |

## RAG API

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/v1/rag/{name}/retrieve` | Retrieve top-k. |
| `POST` | `/v1/rag/{name}/index` | Trigger an index refresh. |
| `GET` | `/v1/rag/sources/{id}/status` | Knowledge source sync status. |

## Approval API

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/v1/approvals` | Create an approval. |
| `POST` | `/v1/approvals/{id}/decide` | Decide. |
| `POST` | `/v1/approvals/{id}/override` | Break-glass override. |

## AgBOM API

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/v1/agbom/{agent}?format=cyclonedx|spdx|swid` | Get the AgBOM. |
| `POST` | `/v1/agbom/{agent}/refresh` | Force refresh. |
| `GET` | `/v1/agbom/{agent}/diff?from={gen}&to={gen}` | Diff two generations. |

## Trace API

The trace ingest is OTLP (HTTP and gRPC). The query API:

| Method | Path | Purpose |
|---|---|---|
| `GET` | `/v1/traces/{id}` | Full trace. |
| `GET` | `/v1/traces?selector=...` | List traces by tag. |
| `GET` | `/v1/spans?execution={id}` | Spans for an execution. |
| `GET` | `/v1/metrics?query=...` | Aggregated metrics. |

## AOS hook surface

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/v1/hooks/aos` | Single endpoint receiving AOS JSON-RPC `steps/*`, `protocols/MCP`, and A2A hook calls. |

The Guardian Agent (default: Joch's policy engine) listens on this endpoint. See [Hooks](../aos/hooks.md).

## Authentication

```text
Authorization: Bearer <oidc-id-token>
```

For service-to-service, Joch issues short-lived workload identities; tokens are 5-minute JWTs.

## Pagination

```text
?limit=100&continue=<token>
```

`WATCH` streams support `?resourceVersion=<rv>` to resume.

## Errors

```json
{
  "error": {
    "code": "PolicyDenied",
    "message": "Policy external-send-requires-approval@v3 denied steps/toolCallRequest",
    "details": {
      "policyId": "external-send-requires-approval",
      "policyVersion": "v3",
      "rule": "tool.sideEffect=external_write requires approval"
    }
  }
}
```

Error codes match the events in [Trace](../specs/kubernetes/trace.md).

## Full OpenAPI / Proto

The full schemas live in [`peasantsai/joch-spec`](https://github.com/peasantsai/joch-spec) under `api/openapi/` and `api/proto/`.
