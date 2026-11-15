# CLAUDE.md

smaltworks is the centralized direction for the smalt projects: the goal, the project-wide decisions and ADRs, the standards
every repository follows, and the exceptions to them. Start with [README.md](README.md) and [CONTEXT.md](CONTEXT.md).

## Rules for this repository

- **Direction, not brand.** Names, voice, marks, colours, the site kit and smaltworks.dev belong to smalt-brand
  ([docs/brand-and-works.md](docs/brand-and-works.md)).
- **Record decisions.** A new or changed rule goes into [DECISIONS.md](DECISIONS.md) with its date and reason, and
  [STANDARDS.md](STANDARDS.md) shows the result. Keep earlier decisions; mark what they were superseded by.
- **Exceptions are explicit.** A repository that departs from a standard on purpose gets a row in
  [EXCEPTIONS.md](EXCEPTIONS.md), with who approved it and when it ends.
- **ADRs:** tier 1 here in `adr/`, tier 2 in their repositories; one numbering for both, and every record gets a row in
  [adr/README.md](adr/README.md).
- **Copies stay unchanged.** Files under `adr/imported/` are copies of records in other repositories, with their source
  noted at the top. Don't edit them; a change to one becomes a new ADR, or an amendment in a PR that also updates the
  source until it is removed.
- **Ask before changing another repository.** Changes smaltworks implies for other repositories are proposed first (in
  `reviews/` or in the conversation) and made only after the owner's yes.
- **No standing monitoring.** No scheduled checks, polling or automation that spends tokens to watch for work. Reviews
  run when the owner asks for one.

## Working rules for sessions

The family's session rules apply here too: [context/session-rules.md](context/session-rules.md).
