# Get Started

Joch is installable in minutes and useful in an hour. The pages below take you from zero to a Joch-governed agent calling MCP tools through the gateway, with a portable AgBOM and approvals enabled.

<div class="grid cards" markdown>

-   :material-numeric-1-circle-outline: **Install**

    ---

    Install Joch locally, in Docker Compose, or on Kubernetes.

    [Install Joch](install.md)

-   :material-numeric-2-circle-outline: **First Agent**

    ---

    Apply your first `Agent` record and run it through the gateway with a `Policy` and an `AgBOM`.

    [Build your first agent](first-agent.md)

-   :material-numeric-3-circle-outline: **Govern an MCP Server**

    ---

    Register, pin, scan, and quarantine an MCP server in 10 minutes.

    [Govern an MCP server](governing-mcp.md)

-   :material-numeric-4-circle-outline: **Onboard from a Vendor SDK**

    ---

    Bring an existing OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, or CrewAI app under Joch governance.

    [Migrate from a vendor SDK](migration-from-vendor-sdk.md)

</div>

## What you need

- macOS, Linux, or Windows (WSL2).
- Docker (optional but recommended for the Compose path).
- A Kubernetes cluster (only for the cluster install).
- An API key for at least one model provider — OpenAI, Anthropic, Google, Microsoft Foundry, or a local model server.

## What you do not need

- Any agent code yet — the quickstart includes a working example.
- A subscription to Joch Cloud (everything below is open source).
- Familiarity with Kubernetes (the local and Compose paths skip it).

## What you get in 30 minutes

- Joch running locally or in Docker Compose.
- One registered `Agent` with a `ModelRoute`, a `Policy`, and an `AgBOM`.
- One governed MCP server with version pinning and trust scoring.
- A working tool call audit trail and a portable conversation log.
- An approval wired to Slack or email (optional).
