# smaltworks

The centralized direction for the smalt projects: one place for the goal, the project-wide decisions and ADRs, the standards
every repository follows, and the exceptions to them.

## The goal

> **Small, Alternative projects supporting small data in the browser, on a unified platform, in a common language:
> TypeScript.**
>
> This doesn't rule out server work; it just isn't the primary focus.
>
> — the owner, 2026-11-15

Every decision, standard and repository is measured against this. Work that doesn't serve it waits.

## What lives here

| File | What it holds |
| --- | --- |
| [CONTEXT.md](CONTEXT.md) | The base context every smalt repository reads first: the goal, the family, where everything is |
| [STANDARDS.md](STANDARDS.md) | The rules every smalt repository follows |
| [EXCEPTIONS.md](EXCEPTIONS.md) | Where a repository departs from a standard, and why |
| [DECISIONS.md](DECISIONS.md) | Project-wide decisions, dated, with their reasons |
| [adr/](adr/README.md) | The ADR index for every repository, the shared numbering, and the project-wide (tier 1) ADRs |
| [WORK.md](WORK.md) | Cross-repository work that is proposed, waiting for decisions, or under way |
| [releases/](releases/) | Release notes, one folder per release |
| [templates/](templates/) | Templates: release notes |
| [context/](context/) | Shared context documents, such as the session rules |
| [docs/brand-and-works.md](docs/brand-and-works.md) | What belongs here and what belongs in smalt-brand |
| [reviews/](reviews/) | Reviews of the repositories against the standards |

## Precedence

smaltworks is the source of project direction. When a repository and smaltworks disagree, smaltworks wins, unless
[EXCEPTIONS.md](EXCEPTIONS.md) records an exception for that repository. A newer decision wins over an older copy
anywhere else.

Brand (name, voice, marks, colours, tokens, badges, the site kit and smaltworks.dev) belongs to
[smalt-brand](https://github.com/stenhoor/smalt-brand). See [docs/brand-and-works.md](docs/brand-and-works.md).

## Migration status

The direction material and the project-wide ADRs were **copied** here on 2026-11-15 from smalt-brand and the product
repositories. Each copy names its source and commit. The originals stay in place until a later PR removes them and
points their repositories here.

## License

[MIT](LICENSE).
