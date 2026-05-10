# Integrations

Joch AI integrates with agent systems at their existing tool, callback, hook, or proxy boundaries.

## MVP adapters

```text
OpenAI Agents SDK tool wrapper
Claude Agent SDK tool wrapper
LangGraph callback/tool wrapper
CrewAI tool wrapper
Generic MCP proxy
Generic HTTP tool proxy
OpenTelemetry exporter
```

## Integration rule

Joch AI should not force teams to rewrite agents.

Adapters should:

```text
register agent metadata
route tool calls through Joch Gateway
forward MCP calls through Joch Gateway
emit execution events
record model/tool cost metadata when available
surface policy denials in the native framework style
```

## Framework notes

| Framework | Integration surface |
|---|---|
| OpenAI Agents SDK | Tool wrappers, lifecycle hooks, MCP configuration, tracing export. |
| Claude Agent SDK | Tool permissions, hooks, MCP, sessions, and permission prompts. |
| LangGraph | Tool nodes, callbacks, graph execution metadata. |
| CrewAI | Tool wrappers, task execution callbacks, crew metadata. |
| Generic HTTP/MCP | Proxy mode when no framework adapter exists. |

## Provider neutrality

The MVP records model identity for inventory and ABOM purposes. It does not need to be a full model router to prove the gateway wedge.
