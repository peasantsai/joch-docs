# Project Governance

Joch is a [PeasantsAI](https://github.com/peasantsai) project. Day-to-day technical decisions are made by maintainers; major changes go through a public RFC process; the trademark is held by PeasantsAI to keep the canonical project unambiguous.

## Maintainership

Each repository under `peasantsai/joch*` has a maintainers file (`MAINTAINERS.md`) listing current maintainers, their areas of focus, and their contact handles.

Maintainer responsibilities:

- triage incoming issues weekly,
- review and merge PRs,
- shepherd RFCs through public comment,
- represent the repository at community calls.

New maintainers are nominated by existing maintainers, with public announcement and a comment window before confirmation.

## Steering

A neutral **Joch Steering Committee** sets cross-repo direction:

- the canonical roadmap (the public version of [Roadmap](../business/roadmap.md) lives here),
- the boundary between OSS and commercial offerings,
- the relationship with standards bodies (OWASP AOS, MCP, OpenTelemetry GenAI),
- conflict resolution between repositories or maintainers.

The committee includes PeasantsAI-employed and independent members. Membership is announced publicly. Quarterly minutes are published.

## RFC process

Significant changes go through an RFC in [`peasantsai/joch-spec`](https://github.com/peasantsai/joch-spec):

1. Author opens a PR adding `rfcs/####-title.md`.
2. Maintainers and community comment for at least 14 days (or longer for cross-repo changes).
3. The proposing repo's maintainers (or the steering committee, if cross-repo) accept, request changes, or reject.
4. Accepted RFCs are merged and tracked to implementation in linked issues.

Templates and example RFCs live in `rfcs/0000-template.md`.

## License and CLA

- Apache-2.0 across all open-source repos.
- Contributing requires accepting the Apache-2.0 license terms by submitting the PR. There is no separate CLA.

## Trademark policy

**Joch**, **Joch Cloud**, **Joch Enterprise**, **Joch Console**, **Joch Operator**, **Joch Registry**, and **Joch Marketplace** are trademarks of PeasantsAI. Forks of the open-source project are allowed under Apache-2.0 but may not use the trademarks in branding or marketing.

The detailed [Trademark Policy](#) is published separately and updated as the project grows.

## Releases

- Each repository has its own release cadence and `CHANGELOG.md`.
- The control-plane core (`peasantsai/joch`) targets monthly minor releases and patch releases as needed.
- Adapters target releases tied to upstream SDK / provider versions.
- A canonical compatibility matrix lists supported core / adapter / spec versions.

## Public communication channels

- GitHub Issues and Discussions on each repository.
- Slack / Discord (linked in repository READMEs).
- Quarterly community call (recorded; agenda public).
- Mailing list announcements at major releases.

## Conflict resolution

Conflicts that maintainers cannot resolve escalate to the steering committee. The committee decides by simple majority. Decisions and rationale are recorded in the meeting minutes.

## Joining as a vendor or partner

Vendors interested in shipping their MCP servers, tools, or framework adapters as official entries should:

1. Open a registry PR with a draft entry,
2. Engage with the relevant repository's maintainers,
3. Pass the conformance suite where applicable.

Vendors do not need to be PeasantsAI customers to ship in the open registry.

## Honest summary

The project is run as an open-source project, with a commercial entity behind it that benefits from broad, well-governed adoption. The governance model prioritizes clarity, accountability, and durability of the canonical project — not control over contributors' work.
