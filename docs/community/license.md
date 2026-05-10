# License

Joch's open-source code is released under the [Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0). The license applies across all `peasantsai/joch*` repositories under PeasantsAI, with the exception of `peasantsai/joch-cloud`, which is a private repository for the commercial multi-tenant control plane.

## Apache-2.0 in plain language

You may:

- use Joch for any purpose, including commercial,
- modify Joch and distribute your modified version,
- include Joch in your own product (subject to attribution and patent terms in the license),
- redistribute the source or compiled binaries.

You must:

- preserve the Apache-2.0 license notice and a copy of the license,
- preserve copyright notices, attribution notices, and the `NOTICE` file,
- state any modifications you make in modified files,
- comply with the Apache-2.0 patent grant terms.

You may not:

- use the **Joch** name or trademarks in branding or marketing for forks (see [Project Governance](governance.md)).

This is not legal advice. The full license text governs.

## Why Apache-2.0

- Industry standard for control-plane projects (Kubernetes, Helm, OpenTelemetry, OPA).
- Permissive enough that enterprise procurement does not block adoption.
- Patent grant provides downstream users protection.
- Compatible with downstream commercial use, including SaaS.

## Why not BSL / SSPL / "source-available"

We deliberately do not use Business Source License, Server Side Public License, or "source-available" custom licenses. Those licenses change adoption dynamics, frighten enterprise procurement, and tend to fragment communities. The cost of the additional revenue protection they provide is, in our analysis, higher than the cost of a generous Apache-2.0 with a strong trademark.

## Per-component licenses

| Component | License |
|---|---|
| `peasantsai/joch` (core) | Apache-2.0 |
| `peasantsai/joch-spec` | Apache-2.0 |
| `peasantsai/joch-docs` | Apache-2.0 (text); CC-BY-4.0 (some images, where noted) |
| `peasantsai/joch-examples` | Apache-2.0 |
| `peasantsai/joch-registry` | Apache-2.0 (registry framework); per-entry licenses noted in entries |
| `peasantsai/joch-operator` | Apache-2.0 |
| `peasantsai/joch-mcp` | Apache-2.0 |
| `peasantsai/joch-js`, `joch-python` | Apache-2.0 |
| `peasantsai/joch-ui` | Apache-2.0 |
| `peasantsai/joch-cloud` | Proprietary (commercial) |
| Joch Enterprise distribution | Commercial license |

## Third-party dependencies

Each repository ships an `LICENSES/` directory and a SBOM in CycloneDX. Contributors must use the dependency-allow-list to avoid introducing copyleft (GPL / AGPL) dependencies into the core, since these would propagate license obligations to downstream users.

## Trademark

Apache-2.0 does not grant trademark rights. **Joch**, **Joch Cloud**, **Joch Enterprise**, **Joch Console**, **Joch Operator**, **Joch Registry**, and **Joch Marketplace** are trademarks of PeasantsAI. The [Trademark Policy](#) is published separately.

## Questions

License or trademark questions can be sent to `legal@peasantsai.io`. For contribution license questions, see [Contributing](contributing.md).
