# Services

## `joch-server`

Responsibilities:

```text
API server
resource registry
agent registry
tool registry
MCP registry
policy storage
approval storage
audit/event log
```

The server is the system of record. Agents, tools, MCP servers, policies, tool calls, approvals, executions, trace events, and ABOMs are all queryable through it.

## `joch-gateway`

Responsibilities:

```text
MCP/tool proxy
tool-call authorization
approval pause/resume
argument redaction
result filtering
audit logging
```

The gateway is the enforcement point. It sees requests before tools run and results before agents receive them.

## `joch-worker`

Responsibilities:

```text
agent discovery
MCP scanning
ABOM generation
background indexing
event processing
```

The worker keeps inventory and derived records current without putting long-running scans on request paths.

## `joch` CLI

Responsibilities:

```text
developer interface
apply/get/describe commands
gateway control
policy application
ABOM generation
trace inspection
```

The CLI is the primary MVP operator interface. A UI can come later, after the command and API model are stable.
