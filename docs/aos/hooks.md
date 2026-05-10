# Hooks (Instrument)

Hooks are the **instrument** pillar of OWASP AOS. They are synchronous interception points in the agent loop where a Guardian Agent receives a payload and returns one of three decisions:

| Decision | Behavior |
|---|---|
| `allow` | Continue with the original request. |
| `deny` | Block the request; the agent receives a denial reason. |
| `modify` | Continue with the `modifiedRequest` provided by the Guardian. |

Joch's gateways implement every standard AOS hook. The default Guardian Agent is Joch's [policy engine](../architecture/policy-engine.md); third-party Guardian Agents can substitute or compose.

## Hook surface

| Hook | Method | Where Joch enforces it |
|---|---|---|
| Agent Trigger | `steps/agentTrigger` | [Model Router](../architecture/model-router.md) on execution start |
| Tool Call Request | `steps/toolCallRequest` | [Tool Gateway](../architecture/tool-gateway.md), [MCP Gateway](../architecture/mcp-gateway.md) |
| Tool Call Result | `steps/toolCallResult` | [Tool Gateway](../architecture/tool-gateway.md), [MCP Gateway](../architecture/mcp-gateway.md) |
| User Message | `steps/message` (role: user) | [Model Router](../architecture/model-router.md) |
| Agent Response | `steps/message` (role: agent) | [Model Router](../architecture/model-router.md) |
| Memory Context Retrieval | `steps/memoryContextRetrieval` | [Memory service](../architecture/data-plane.md) |
| Memory Store | `steps/memoryStore` | [Memory service](../architecture/data-plane.md) |
| Knowledge Retrieval | `steps/knowledgeRetrieval` | [RAG service](../architecture/data-plane.md) |
| MCP Outbound | `protocols/MCP` (outbound) | [MCP Gateway](../architecture/mcp-gateway.md) |
| MCP Inbound | `protocols/MCP` (inbound) | [MCP Gateway](../architecture/mcp-gateway.md) |
| A2A Send | A2A `send_message` | A2A broker |
| A2A Stream | A2A `stream_message` | A2A broker |
| A2A Cancel | A2A `cancel_request` | A2A broker |
| A2A Get Task | A2A `get_task` | A2A broker |
| A2A Notification Config | A2A get/set/resubscribe | A2A broker |

## Wire format

Hook calls are JSON-RPC 2.0. Each request carries a `method` (e.g., `steps/toolCallRequest`), an `id`, and a `params` object containing:

- the action-specific payload (`trigger`, `toolCallRequest`, `toolCallResult`, `message`, `memory`, `knowledgeStep`),
- a `reasoning` field for the agent's stated rationale,
- a `context` block with `agent`, `session`, `turnId`, `stepId`, `timestamp`, and optional `user` / `organization`.

The Guardian responds with an `AOSSuccessResponse` containing `decision` (`allow` / `deny` / `modify`), an optional `reason`, and (for `modify`) a `modifiedRequest`.

## Example: tool call request

```json
{
  "jsonrpc": "2.0",
  "method": "steps/toolCallRequest",
  "id": "13fa8d6f-8f9f-4d01-ba6b-db99d84d77de",
  "params": {
    "toolCallRequest": {
      "executionId": "69dbf4c3-be33-4694-a9f0-8d3a824c5d5b",
      "toolId": "c264f381-10cf-4403-bd11-383014c0fcc6",
      "inputs": [
        { "name": "subject", "value": "Refund request for order 12345" },
        { "name": "body",    "value": "Customer reports the item never arrived." }
      ]
    },
    "reasoning": "Customer is eligible for refund. I will create a Zendesk ticket.",
    "context": {
      "agent": {
        "id": "support-triage",
        "name": "Support Triage",
        "version": "14",
        "provider": { "name": "OpenAI", "url": "https://openai.com/" }
      },
      "session": { "id": "84c36ebb-83aa-4bc9-8670-7aba4cedc70f" },
      "turnId": "69ef57b8-3993-440d-9493-523914f3f149",
      "stepId": "9263448a-186a-4c3b-abcf-443feb44a01e",
      "timestamp": "2026-05-10T10:34:55.123Z"
    }
  }
}
```

A `deny` response from the Guardian:

```json
{
  "jsonrpc": "2.0",
  "id": "13fa8d6f-8f9f-4d01-ba6b-db99d84d77de",
  "result": {
    "decision": "deny",
    "reason": "external_write requires approval; approval not granted",
    "policyId": "external-send-requires-approval",
    "policyVersion": "v3"
  }
}
```

A `modify` response:

```json
{
  "jsonrpc": "2.0",
  "id": "13fa8d6f-8f9f-4d01-ba6b-db99d84d77de",
  "result": {
    "decision": "modify",
    "modifiedRequest": {
      "inputs": [
        { "name": "subject", "value": "Refund request for order 12345" },
        { "name": "body",    "value": "[REDACTED-PII] reported the item never arrived." }
      ]
    },
    "reason": "redacted PII in tool argument"
  }
}
```

## Where Joch records the decision

Every hook decision is recorded as a `HookDecision` event in the [Trace](../specs/kubernetes/trace.md) and reflected on the corresponding [`ToolCall`](../specs/kubernetes/toolcall.md), [`Approval`](../specs/kubernetes/approval.md), or other runtime resource. Auditors can replay any execution and see, at every hook boundary, which Guardian decided what, by which policy version, on which inputs.

## Composition

Multiple Guardian Agents can be composed:

- any `deny` is a `deny`,
- `modify` outputs are merged in declared order,
- `allow` is the unanimous outcome.

This makes separation of duties practical: a security team's Guardian runs alongside the platform's policy engine without entangling rules.
