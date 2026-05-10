# Contributing

Joch is an open-source project under [PeasantsAI](https://github.com/peasantsai). Contributions of every shape are welcome — bug reports, documentation, framework adapters, provider adapters, MCP server entries, policies, examples, and core code.

## Where to start

| If you want to … | Start here |
|---|---|
| Report a bug | [`peasantsai/joch` issues](https://github.com/peasantsai/joch/issues) |
| Improve docs | [`peasantsai/joch-docs` issues](https://github.com/peasantsai/joch-docs/issues) |
| Propose a design change | An RFC PR in [`peasantsai/joch-spec`](https://github.com/peasantsai/joch-spec) |
| Add a framework adapter | A package PR in [`peasantsai/joch`](https://github.com/peasantsai/joch) under `pkg/adapters/framework/` |
| Add a provider adapter | A package PR under `pkg/adapters/provider/` |
| Contribute a registry entry | [`peasantsai/joch-registry`](https://github.com/peasantsai/joch-registry) |
| Add an example | [`peasantsai/joch-examples`](https://github.com/peasantsai/joch-examples) |

## Issue triage

- All issues are public.
- A maintainer triages new issues weekly: labels, priority, and roadmap signal.
- "Good first issue" and "help wanted" labels mark welcoming entry points.

## Pull requests

- Fork the relevant repo, branch, push, open a PR.
- One logical change per PR. Smaller is better.
- Include test coverage where applicable. Adapter PRs run the [adapter conformance suite](../specs/kubernetes/framework-adapter.md#conformance) before merge.
- Sign commits where possible. The CLA is Apache-2.0; by submitting a PR you agree to license your contribution under it.

## RFC process

For significant changes (new resource kinds, breaking schema changes, new gateways), open an RFC in [`peasantsai/joch-spec`](https://github.com/peasantsai/joch-spec) under `rfcs/`. RFCs follow the standard pattern:

```text
0000-template.md
0001-resource-model.md
0002-execution-model.md
0003-portable-policy.md
0004-abom.md
0005-aos-conformance.md
```

Each RFC documents motivation, design, alternatives considered, and migration. RFCs go through a public comment window before merge.

## Code style

- Go: `gofmt`, `goimports`, `golangci-lint` clean.
- Python: `ruff`, `black` clean; type hints required for public APIs.
- TypeScript: `prettier`, `tsc` strict, `eslint` clean.
- Markdown: one sentence per line where reasonable; preferred line length 100; lists use `-`.

## Tests

- Unit tests for new functions and methods.
- Integration tests for new resource kinds, against a live or `testcontainers`-managed control plane.
- Adapter PRs must pass the conformance suite end-to-end against a sample agent.

## Documentation

- Every new resource kind, gateway, or service ships a doc page in `peasantsai/joch-docs` in the same PR (cross-repo if needed).
- Every breaking change updates the [Roadmap](../business/roadmap.md), the relevant resource page, and any affected examples.

## Code of conduct

We follow the [Contributor Covenant](https://www.contributor-covenant.org/). Be kind, assume good intent, and prefer clear technical reasoning over heat. Maintainers reserve the right to remove comments and contributions that violate the code.

## Recognition

- All contributors are listed in `CONTRIBUTORS.md` per repo.
- Significant adapter contributions are highlighted in release notes.
- Sustained contributors can be nominated for maintainer status. See [Project Governance](governance.md).

## Help and support

- GitHub Discussions for general questions.
- Slack / Discord (see the README of each repo) for synchronous chat.
- Maintainer office hours announced on the website monthly.

Welcome aboard.
