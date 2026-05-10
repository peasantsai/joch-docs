# Claude Agent SDK Integration

The [**Claude Agent SDK**](https://code.claude.com/docs/en/agent-sdk/overview) (formerly Claude Code SDK) gives you Claude Code as a library: built-in tools, agent loop, hooks, subagents, MCP, permissions, sessions, skills, slash commands, plugins. Joch attaches to it through the `claude-agent-sdk` framework adapter.

## The SDK in 60 seconds

| Primitive | Role |
|---|---|
| `query(*, prompt, options=None, transport=None)` | Async iterator returning `Message` objects from the agent loop. |
| `ClaudeAgentOptions` | Configuration dataclass with `tools`, `allowed_tools`, `disallowed_tools`, `system_prompt`, `mcp_servers`, `plugins`, `permission_mode`, `can_use_tool`, `continue_conversation`, `resume`, `fork_session`, `max_turns`, `max_budget_usd`, `agents`, `hooks`, `include_hook_events`, `settings`, `setting_sources`, `cwd`, `add_dirs`, `env`, `extra_args`, `model`, `fallback_model`, `betas`, `output_format`, `thinking`, `effort`, `enable_file_checkpointing`, `user`, `include_partial_messages`, `session_store`, `sandbox`. |
| `ClaudeSDKClient` | Persistent session client with `connect`, `query`, `receive_messages`, `receive_response`, `interrupt`, `set_permission_mode`, `set_model`, `rewind_files`, `get_mcp_status`, `reconnect_mcp_server`, `toggle_mcp_server`, `stop_task`, `get_server_info`, `disconnect`. |
| Built-in tools | `Read`, `Write`, `Edit`, `Bash`, `Monitor`, `Glob`, `Grep`, `WebSearch`, `WebFetch`, `AskUserQuestion`, plus `Agent` (subagent invocation). |
| Hooks | `PreToolUse`, `PostToolUse`, `PrePermissionRequest`, `PostPermissionRequest`, `Stop`, `SessionStart`, `SessionEnd`, `UserPromptSubmit`, …; bound via `HookMatcher(event, handler, filter)`. |
| Subagents (`agents={...}`) | `AgentDefinition(description, prompt, tools, disallowedTools, model, skills, memory, mcpServers, initialPrompt, maxTurns, background, effort, permissionMode)`. |
| Permission modes | `default`, `acceptEdits`, `plan`, `dontAsk`, `bypassPermissions`. |
| Setting sources | `user` (`~/.claude/settings.json`), `project` (`.claude/settings.json`), `local` (`.claude/settings.local.json`). |
| Skills, slash commands, memory, plugins | Filesystem-based: `.claude/skills/*/SKILL.md`, `.claude/commands/*.md`, `CLAUDE.md`, programmatic `plugins=[...]`. |
| Messages | `UserMessage`, `AssistantMessage`, `SystemMessage`, `ResultMessage`, `StreamEvent`, `RateLimitEvent`. Content blocks: `TextBlock`, `ThinkingBlock`, `ToolUseBlock`, `ToolResultBlock`. |
| Provider auth | Anthropic API keys; `CLAUDE_CODE_USE_BEDROCK=1`, `CLAUDE_CODE_USE_VERTEX=1`, `CLAUDE_CODE_USE_FOUNDRY=1`. |

## What Joch wraps and what it does not

```text
                ┌───────────────── your code ─────────────────┐
                │   from claude_agent_sdk import query, ...   │
                │   async for m in query(prompt=..., options): │
                │       ...                                   │
                └─┬───────────────┬───────────────┬───────────┘
                  │ built-in tool │ MCP servers   │ hooks (PreToolUse, ...)
                  ▼               ▼               ▼
   ── Joch boundary ─────────────────────────────────────────────
   Bash / Read / Edit / etc. │ MCP traffic    │ hook events
                  ▼              ▼               ▼
         ┌──────────────────────────────────────────────┐
         │ Joch claude-agent-sdk FrameworkAdapter       │
         │   intercepts built-in tool calls             │
         │   wraps mcp_servers map with MCP Gateway     │
         │   routes provider calls through Model Router │
         │   bridges sessions to Conversation           │
         │   maps PreToolUse to AOS toolCallRequest     │
         │   forwards messages to Joch Trace (OTLP)     │
         │   surfaces deny/modify in PermissionRequest  │
         └──────────────────────────────────────────────┘
```

Joch does **not** replace the Claude Code agent loop, the built-in tool implementations, the SDK's session storage on disk, the SDK hook surface, or the permission UI semantics. Your `query()` calls stay identical.

## Mapping table

| Claude Agent SDK | Joch |
|---|---|
| `query(prompt=..., options=ClaudeAgentOptions(...))` | Drives one execution under an [`Agent`](../specs/kubernetes/agent.md) with `framework.adapterRef: claude-agent-sdk` |
| `allowed_tools=[...]` | Pre-applied at the [Tool Gateway](../architecture/tool-gateway.md); enforced uniformly with cross-SDK [`Policy`](../specs/kubernetes/policy.md) |
| `mcp_servers={"name": {...}}` | One [`MCPServer`](../specs/kubernetes/mcpserver.md) record per server; traffic via the [MCP Gateway](../architecture/mcp-gateway.md) |
| `agents={"reviewer": AgentDefinition(...)}` | Each subagent registered as an `Agent` record with `framework: claude-agent-sdk` and `parent_tool_use_id` linkage in trace events |
| Built-in tools (`Read`, `Edit`, `Bash`, …) | Each fires `steps/toolCallRequest` / `steps/toolCallResult`; PII redaction and side-effect classification applied at gateway |
| `hooks={"PreToolUse": [...]}` | Continue to run; Joch additionally records `HookDecision` events with policy id and version |
| `permission_mode` | Composed with portable `Policy`. `bypassPermissions` is rejected by default in `prod` `Environment` |
| `can_use_tool` callback | Wrapped: Guardian Agent's `deny` / `modify` is propagated into the callback's return value |
| `resume`, `continue_conversation`, `fork_session` | Bridged to [`Conversation`](../specs/kubernetes/conversation.md) and [`StateCheckpoint`](../specs/kubernetes/state-checkpoint.md) |
| `model`, `fallback_model` | Replaced by [`ModelRoute`](../specs/kubernetes/model-route.md) when an agent record references one |
| `SystemMessage` / `AssistantMessage` / `ResultMessage` | Mirrored into trace events with token + cost attribution |
| `setting_sources`, `settings`, `.claude/` files | Captured into the `Agent` record's framework metadata; visible in the [`AgBOM`](../specs/kubernetes/agbom.md) |
| Bedrock / Vertex / Foundry routing | Recognized by the [Model Router](../architecture/model-router.md) and reflected in `Trace` provider attribution |

## The smallest install

### Before — your existing code

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions

async def main():
    async for message in query(
        prompt="Find and fix the bug in auth.py",
        options=ClaudeAgentOptions(allowed_tools=["Read", "Edit", "Bash"]),
    ):
        print(message)

asyncio.run(main())
```

### After — Joch-governed

```python
import asyncio
from claude_agent_sdk import query, ClaudeAgentOptions
from joch.adapters.claude_agent_sdk import joch_options  # adapter package

async def main():
    options = joch_options(
        ClaudeAgentOptions(allowed_tools=["Read", "Edit", "Bash"]),
        joch_agent_ref="bug-fixer",
        server_url="http://joch-server:8080",
    )
    async for message in query(
        prompt="Find and fix the bug in auth.py",
        options=options,
    ):
        print(message)

asyncio.run(main())
```

`joch_options(...)` does five things:

- attaches a `PreToolUse` hook that emits `steps/toolCallRequest` to the Joch policy engine and applies the returned decision (`allow` / `deny` / `modify`),
- attaches a `PostToolUse` hook that emits `steps/toolCallResult` and applies the result decision,
- replaces `mcp_servers` entries with gateway-routed configs whose names are validated against registered [`MCPServer`](../specs/kubernetes/mcpserver.md) records,
- routes `query()` provider calls through the Joch model router (preserving Bedrock / Vertex / Foundry env vars),
- writes every `AssistantMessage`, `ToolUseBlock`, `ToolResultBlock`, and `ResultMessage` into Joch trace.

## Hooks → AOS hooks

| Claude Agent SDK hook | AOS hook |
|---|---|
| `UserPromptSubmit` | `steps/agentTrigger` |
| `PreToolUse` | `steps/toolCallRequest` |
| `PostToolUse` | `steps/toolCallResult` |
| `PrePermissionRequest` | composes with Joch `Approval` flow |
| `PostPermissionRequest` | records the operator decision into `Approval.status` |
| `SessionStart` | `ExecutionCreated` / `ExecutionStarted` |
| `SessionEnd` | `ExecutionSucceeded` / `ExecutionFailed` |
| `Stop` | `Execution.status.phase: Cancelled` |

Decisions returned by your hooks are honored. Joch composes its policy decisions with yours: any `deny` is a `deny`; `modify` outputs are merged in declared order.

## Subagents

`agents={"reviewer": AgentDefinition(...)}` registers a subagent. The Joch adapter:

- creates a child `Agent` record at apply time (or at first invocation, with sensible defaults),
- links its executions to the parent via `parent_tool_use_id` and `joch.parent.execution.id` in the trace,
- enforces the same policies on subagent tool / MCP / model calls as on the parent.

This makes "the bug-fixer agent invoked the code-reviewer subagent" auditable across the boundary.

## Settings sources and the AgBOM

`setting_sources=["user", "project", "local"]` resolves on disk; the resolved set (skills, commands, memory, plugins) is captured into the `Agent` record and surfaces in the [`AgBOM`](../specs/kubernetes/agbom.md) as components:

- `urn:claude:skill:<name>@<sha>` for `.claude/skills/<name>/SKILL.md`,
- `urn:claude:slash-command:<name>@<sha>` for `.claude/commands/<name>.md`,
- `urn:claude:plugin:<name>@<version>` for programmatic `plugins=[...]`,
- `urn:claude:memory:CLAUDE.md@<sha>` for project memory.

Operators can require a minimum trust level for any of these via `Policy`.

## Sessions

`continue_conversation`, `resume=<session_id>`, and `fork_session=True` are bridged into Joch:

- the SDK's JSONL session file remains the source for in-process replay,
- Joch additionally writes a portable [`Conversation`](../specs/kubernetes/conversation.md) event log,
- a `StateCheckpoint` is generated when the operator invokes `joch agents switch ...` to migrate to a different framework or provider.

## Approvals

`permission_mode="default"` plus a Joch `Policy` that requires `human` approval for `external_write` tools yields:

```text
PreToolUse → toolCallRequest → Guardian: require approval → suspend tool call
                                                  ↓
                                   Approval record created
                                                  ↓
                                  approver decides via console / Slack
                                                  ↓
                          query() resumes with allow / deny / modify
```

The SDK's `PrePermissionRequest` hook still runs; your in-process logic composes with the Joch flow.

## Tracing

The Claude Agent SDK does not ship a trace exporter equivalent to OTel out of the box. The Joch adapter materializes spans for every:

- `AssistantMessage` → `ModelCallCompleted`,
- `ToolUseBlock` → `ToolCallRequested`,
- `ToolResultBlock` → `ToolCallCompleted`,
- `ThinkingBlock` (if present) → annotation on the `ModelCall` span,
- subagent invocation → child `AgentRun` span linked by `parent_tool_use_id`.

`ResultMessage.usage` populates `gen_ai.usage.input_tokens` and `gen_ai.usage.output_tokens` plus cache attributes.

## Migration matrix

| Existing | Joch step |
|---|---|
| `query()` calls in scripts and services | Wrap options with `joch_options(...)`; no other code change |
| `mcp_servers={...}` entries | Register matching [`MCPServer`](../specs/kubernetes/mcpserver.md) records; the adapter swaps configs |
| Subagents via `agents={...}` | Register child `Agent` records; conformance tests verify subagent identity links |
| `permission_mode="bypassPermissions"` | Replaced by a `Policy` rule that requires Approval in `prod` and is a hard error in `prod` `Environment` |
| Bedrock / Vertex / Foundry routing | Continue to use the env vars; the model router records the resolved provider |
| `.claude/` settings | Included in the `Agent` record's framework section; visible in `AgBOM` |

## What Joch leaves to the SDK

- The Claude Code agent loop itself.
- The implementation of every built-in tool (`Read`, `Edit`, `Bash`, etc.).
- File checkpointing (`enable_file_checkpointing`).
- The interactive permission UI for the CLI.
- The `Plan` permission mode's read-only enforcement.
- The local JSONL session file format.

## Reference

- Overview: <https://code.claude.com/docs/en/agent-sdk/overview>
- Python reference: <https://code.claude.com/docs/en/agent-sdk/python>
- TypeScript reference: <https://code.claude.com/docs/en/agent-sdk/typescript>
- Hooks: <https://code.claude.com/docs/en/agent-sdk/hooks>
- Subagents: <https://code.claude.com/docs/en/agent-sdk/subagents>
- Permissions: <https://code.claude.com/docs/en/agent-sdk/permissions>
- Sessions: <https://code.claude.com/docs/en/agent-sdk/sessions>
- MCP: <https://code.claude.com/docs/en/agent-sdk/mcp>
- Migration from old SDK: <https://code.claude.com/docs/en/agent-sdk/migration-guide>
