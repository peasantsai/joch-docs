# Joch Docs

Documentation site for **[Joch](https://peasantsai.github.io/joch-docs/)** — the vendor-neutral control plane for AI agent fleets.

Joch operates agents built with OpenAI Agents SDK, Claude Agent SDK, Google ADK, Microsoft Agent Framework, LangGraph, CrewAI, or custom code. It owns the inventory, governance, portability, observability, and release lifecycle that vendor SDKs leave to operators, and it implements the [OWASP Agent Observability Standard](https://aos.owasp.org/) for AgBOM, hooks, and trace.

## Site contents

- **Concepts** — positioning, the five pillars, comparison vs vendor SDKs, glossary.
- **Pillars** — inventory, governance, portability, observability, release management.
- **Architecture** — control plane, data plane, framework adapters, tool gateway, MCP gateway, model router, policy engine, state portability, trust model.
- **Resources** — the Kubernetes-style YAML resource catalog (`Agent`, `FrameworkAdapter`, `Tool`, `MCPServer`, `Policy`, `AgBOM`, `Trace`, `ModelRoute`, and the rest).
- **AOS Conformance** — AgBOM, hooks, events, and the CycloneDX / OpenTelemetry / OCSF mappings.
- **Use Cases** — fleet inventory, MCP governance, cross-provider migration, cost control, release gates, approvals.
- **Business** — product, moat, competitive landscape, revenue models, target audience, go-to-market, open-core strategy, roadmap.
- **Platform** — Kubernetes role, CRDs and controllers, deployment modes, product strategy.
- **Product** — brand, repository strategy.
- **Community** — contributing, governance, license.

## Building locally

```bash
python3 -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
mkdocs serve -f mkdocs.yaml
```

Strict build (used in CI):

```bash
mkdocs build -f mkdocs.yaml --strict
```

## Deployment

The site deploys to GitHub Pages via `.github/workflows/pages.yml` on every push to `main`. Production URL: <https://peasantsai.github.io/joch-docs/>.

## License

Apache-2.0 for content and code; CC-BY-4.0 for designated images. See [community/license.md](docs/community/license.md).
