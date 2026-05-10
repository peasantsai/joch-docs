# Joch AI Docs

Documentation site for **[Joch AI](https://peasantsai.github.io/joch-docs/)**, the open MCP and tool-governance gateway for autonomous agent workforces.

Joch AI does not replace OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, CrewAI, or custom agent code. It sits around them as the system of record, policy enforcement layer, approval point, and audit trail for agent tools and MCP servers.

## Site contents

- **Get Started** - install Joch AI, register a first agent, and govern an MCP server.
- **Strategy** - brand, positioning, market context, business model, and open-source split.
- **MVP** - scope, user stories, roadmap, and success metrics for MVP v0.1.
- **Architecture** - server, gateway, worker, policy, approvals, data model, deployment, and adapters.
- **Resources** - Kubernetes-style YAML specs for Agent, Tool, MCPServer, Policy, ToolCall, Approval, ABOM, Execution, and TraceEvent.
- **Reference** - CLI, REST API, and repository plan.
- **Community** - contribution, license, and FAQ pages.

## Building locally

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
mkdocs serve -f mkdocs.yaml
```

Strict build, matching CI:

```bash
mkdocs build -f mkdocs.yaml --strict
```

## Deployment

The site deploys to GitHub Pages through `.github/workflows/pages.yml` on every push to `main`. Production URL: <https://peasantsai.github.io/joch-docs/>.

## License

Apache-2.0 for content and code. See [community/license.md](docs/community/license.md).
