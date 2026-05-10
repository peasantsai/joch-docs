# API Reference

The MVP API is REST-first.

## Agent API

```text
GET  /v1/agents
POST /v1/agents
GET  /v1/agents/{name}
GET  /v1/agents/{name}/abom
```

## Tool API

```text
GET  /v1/tools
GET  /v1/tools/{name}
GET  /v1/toolcalls
POST /v1/toolcalls
```

## MCP API

```text
GET  /v1/mcpservers
POST /v1/mcpservers/scan
GET  /v1/mcpservers/{name}/tools
```

## Policy API

```text
GET  /v1/policies
POST /v1/policies
POST /v1/policy/evaluate
```

## Approval API

```text
GET  /v1/approvals
POST /v1/approvals/{id}/approve
POST /v1/approvals/{id}/reject
```

## Trace/Event API

```text
GET  /v1/executions
GET  /v1/executions/{id}
GET  /v1/executions/{id}/events
GET  /v1/events
```

## Authentication

MVP local mode can run without auth. Team and production modes should use OIDC bearer tokens and service-to-service credentials.

```text
Authorization: Bearer <token>
```

## Error shape

```json
{
  "error": {
    "code": "PolicyDenied",
    "message": "tool shell.exec is denied by policy external-write-approval",
    "details": {
      "policy": "external-write-approval",
      "rule": "tool.name=shell.exec",
      "toolCall": "call-123"
    }
  }
}
```
