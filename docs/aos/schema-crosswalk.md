# AOS Schema Crosswalk

The OWASP AOS specification publishes a JSON Schema with formal definitions for every entity in the standard. This page maps each AOS schema definition to its Joch counterpart so that downstream tooling (Guardian Agents, BOM consumers, SIEMs) can interoperate without bespoke integration.

The authoritative AOS schema lives at:

- `https://raw.githubusercontent.com/OWASP/www-project-agent-observability-standard/dev/specification/AOS/aos_schema.json`

## Identity and provenance

| AOS definition | Joch counterpart | Notes |
|---|---|---|
| `Agent` | [`Agent`](../specs/kubernetes/agent.md) record | The Joch record carries `framework.adapterRef`, model policy, tool / MCP refs, policies, budgets, observability config. |
| `AgentIdentity` | `Agent.metadata` + `joch.agent.*` trace attributes | Stable identity across executions. |
| `AgentProvider` | `FrameworkAdapter` + `framework` block on `Agent` | Identifies the SDK that runs the agent. |
| `AgentSignature` | `Agent.metadata.annotations["joch.dev/signature"]` + signed `AgBOM` | When ABOM signing is enabled. |
| `AgentTrigger` / `AgentTriggerEvent` / `AgentTriggerStep` | `Execution.spec.trigger` + `agentTrigger` hook | Webhook, manual, scheduled, upstream. |
| `Identity`, `MachineIdentity`, `User`, `Organization` | `Team / Namespace`, `Approval.requestedBy`, audit log identity | Joch tracks operator identity in audit; agent runtime identity in workload tokens. |
| `Session` | [`Conversation`](../specs/kubernetes/conversation.md) | Vendor-neutral, durable. |

## Tools

| AOS definition | Joch counterpart |
|---|---|
| `ToolDefinition` | [`Tool`](../specs/kubernetes/tool.md) record |
| `ToolArgumentDefinition` | `Tool.spec.inputSchema` |
| `ToolOutputDefinition` | `Tool.spec.outputSchema` |
| `ToolCallRequest` | [`ToolCall.spec.input`](../specs/kubernetes/toolcall.md) |
| `ToolCallResult` | [`ToolCall.status.output`](../specs/kubernetes/toolcall.md) |
| `ToolCallRequestStep` (hook payload) | AOS `steps/toolCallRequest` payload at the [Tool Gateway](../architecture/tool-gateway.md) |
| `ToolCallResultStep` (hook payload) | AOS `steps/toolCallResult` payload at the [Tool Gateway](../architecture/tool-gateway.md) |
| `ToolArgumentValue` | input value with `joch.tool.input.<key>` redaction rules from policy |

## Knowledge / RAG / Sources

| AOS definition | Joch counterpart |
|---|---|
| `KnowledgeRetrievalStep` | AOS `steps/knowledgeRetrieval` at the RAG service |
| `KnowledgeRetrievalStepParams` | request payload (`query`, `keywords`) |
| `KnowledgeRetrievalResult` | response payload with results + citations |
| `Resource` | [`KnowledgeSource`](../specs/kubernetes/knowledge-source.md) — points at the corpus |
| `Source`, `SiteSource`, `FileSource` | `KnowledgeSource.spec.source` (`web`, `file`, `s3`, …) |

## Memory

| AOS definition | Joch counterpart |
|---|---|
| `MemoryContextRetrievalStep` | AOS `steps/memoryContextRetrieval` at the [Memory service](../architecture/data-plane.md) |
| `MemoryStoreStep` | AOS `steps/memoryStore` at the Memory service |
| `Memory` content shapes | [`Memory`](../specs/kubernetes/memory.md) record + per-type backend (working / semantic / episodic / procedural / preference / blackboard) |

## Models

| AOS definition | Joch counterpart |
|---|---|
| `Model` | [`Model`](../specs/kubernetes/model.md) record |
| `LlmProvider` | provider field on `Model.spec.provider` and provider adapter |
| `MessageStep` | AOS `steps/message` at the [Model Router](../architecture/model-router.md) |
| `Message` content shape | canonical message format documented in [State Portability](../architecture/state-portability.md) |
| `Part`, `TextPart`, `DataPart`, `FilePart`, `FileWithBytes`, `FileWithUri` | content blocks in the canonical message format |

## MCP

| AOS definition | Joch counterpart |
|---|---|
| `MCPMessage` | request/response payload at the [MCP Gateway](../architecture/mcp-gateway.md) |
| `MCPServer` (AOS schema) | [`MCPServer`](../specs/kubernetes/mcpserver.md) Joch record |

## A2A and Agent-to-Agent

| AOS definition | Joch counterpart |
|---|---|
| `A2AContext`, `A2AFullAgentContext`, `A2APartialAgentContext`, `A2APartialAgentDetails` | `Handoff.spec.context` and `joch.a2a.*` trace attributes |
| `A2AMessageSend` | A2A broker outbound; AOS `send_message` hook |
| `A2AMessageStream` | A2A broker outbound streaming; `stream_message` hook |
| `A2ATasksCancel` | `cancel_request` hook |
| `A2ATaskGet` | `get_task` hook |
| `A2ATaskPushNotificationConfigSet` / `Get` | notification config hooks |
| `A2ATasksResubscribe` | `resubscribe` hook |

## Step / hook envelope

| AOS definition | Joch counterpart |
|---|---|
| `StepContext` | The `context` block in every AOS hook payload (agent, session, turnId, stepId, timestamp, optional user/org). |

## Wire protocol

| AOS definition | Joch counterpart |
|---|---|
| `JSONRPCRequest`, `JSONRPCResponse`, `JSONRPCErrorResponse`, `JSONRPCError`, `MethodNotFoundError`, `InvalidParamsError`, `InvalidRequestError`, `JSONParseError`, `InternalError` | The wire format Joch uses on `/v1/hooks/aos`. Exact JSON-RPC 2.0 conformance. |
| `ASOPRequest`, `ASOPResponse`, `ASOPSuccessResponse`, `ASOPSuccessResult` | The Joch policy engine returns ASOPSuccessResponse from every hook call (`decision`, `reason`, `modifiedRequest`). |
| `PingRequest`, `PingRequestSuccessResponse`, `PingRequestResult` | Health-check JSON-RPC method on the hook endpoint. |

## How to verify conformance

```bash
joch aos verify --agent support-triage
joch aos verify --suite hooks --strict
joch aos verify --suite agbom --format cyclonedx
joch aos verify --suite trace --transport otlp
```

The `joch aos verify` subcommand exercises:

- every AOS hook with a synthetic Guardian,
- AgBOM emission in CycloneDX, SPDX, and SWID,
- trace event taxonomy round-trip,
- and JSON-RPC request / response shape against the schema.

A failed verification names the AOS definition that did not round-trip and the Joch field that was missing or malformed.

## Why this page exists

OWASP AOS is the conformance baseline. Without an explicit crosswalk, customers, auditors, and third-party Guardian Agent vendors have to read both the AOS spec and the Joch resource catalog to confirm interoperability. This page collapses that work into a single table.
