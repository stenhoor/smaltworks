# ADR 0005: The licence

> **Copied** from [smaltfl/docs/adr/0005-licence.md](https://github.com/stenhoor/smaltfl/blob/9535991da5dcf9480f59ced189bde5bec4f13138/docs/adr/0005-licence.md) (commit `9535991`) on 2026-11-15. The source stays in place until its removal PR. Relative links were rewritten to resolve from smaltworks.

- **Status:** accepted
- **Date:** 2026-11-05

## Context

The first code landed without a licence, because a licence is the maintainers' choice (Q-44). Every smalt repository
is MIT, with the copyright held by its contributors (`Copyright (c) 2026 smaltdb contributors`).

## Decision

smaltfl is MIT-licensed, `Copyright (c) 2026 smaltfl contributors`, as the rest of the family is. (Q-44)

## Consequences

- `LICENSE` is in the repository and in the package's files, and `package.json` says `"license": "MIT"`.
- The package stays `private` until its first publish, as `@smaltworks/smaltfl` (ARCHITECTURE.md §11), so it can't be
  published by accident before then.
