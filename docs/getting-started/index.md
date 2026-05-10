# Get Started

Use this section to run Joch AI locally, register a first agent, and put an MCP server behind governance.

The fastest path:

```bash
joch init
joch up
joch discover ./repo
joch get agents
joch mcp scan
joch policy apply -f policies/external-write-approval.yaml
joch gateway start
```

## What you need

- A workstation with Docker available for the Compose path.
- An agent project built with OpenAI Agents SDK, Claude Agent SDK, LangGraph, CrewAI, or custom code.
- At least one tool or MCP server you want Joch AI to inspect, govern, and audit.

## First workflow

1. Install the CLI.
2. Start the local control plane.
3. Discover or register an agent.
4. Scan tool and MCP capabilities.
5. Apply a policy.
6. Route tool calls through Joch Gateway.
7. Inspect tool calls, approvals, traces, and the ABOM.

Continue with [Install](install.md).
