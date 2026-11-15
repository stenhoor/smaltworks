# Drift review, 2026-11-15

A review of the 22 smalt repositories against the rules now collected in smaltworks, from shallow clones of each
repository's default branch on 2026-11-15. Nothing in any other repository has been changed. Every change proposed here
waits for the owner's yes.

## Findings

### Where direction lives

Direction was spread over three places:

- **smalt-brand** held the cross-repository work register (`WORK.md`), the project-wide parts of `DECISIONS.md` and
  `BRAND.md` (TypeScript and browsers, outside code, npm names, plugins, the documentation standard), four project-wide
  ADRs (0024–0026, 0032) and the 0.9.0 release notes. All of it is now copied here.
- **smaltdb's `CLAUDE.md`** is the reference copy of the session rules; other repositories say "follow smaltdb's
  Issues section". Now copied to [context/session-rules.md](../context/session-rules.md).
- **smaltfl's `CLAUDE.md`** (513 lines) carries its own map of the smalt repositories.

### ADRs

- 74 records in 9 repositories. 34 are project-wide (tier 1) and are copied here; see the [index](../adr/README.md).
- The shared sequence (0001–0036) has no gaps and no clashes, except that **ADR 0024 is in two repositories** and
  smalt-brand's copy is out of date.
- **smaltcy and smaltgl** started their own sequences from 0001 after the shared rule (2026-11-09); smaltbi started
  before it; smaltfl has its own by exception.
- **No ADRs at all** in smaltql, smaltdf, smaltkit, smaltnb, smaltui or the smaltdb plugins. Several are the subject of
  other repositories' ADRs, so this may be fine; worth a look for smaltkit, which many records depend on.
- Three formats are in use for the status and date lines. [adr/TEMPLATE.md](../adr/TEMPLATE.md) takes the most common.

### Standards

| Rule | Drift |
| --- | --- |
| npm names: `@smaltworks/<repository>` | Published or publishable under other names: `smaltdb`, `smaltkit`, `smaltnb` (unscoped); `@smaltdb/io`, `@smaltdb/external`, `@smaltdb/parquet`, `@smaltdb/df`. The renames are planned for 0.9.0 (smaltdb ADR 0015 step 11) |
| TypeScript: checking on 7.0, emitting on 6.0.3 for one more release | **Behind:** smaltdb-node and smaltdf are on TypeScript 5.9. **Ahead:** smaltgl is on 7.1.0-beta. **All on 7:** smaltcy, smaltml, smaltbi and smaltui use 7 for everything, including any emitting. **None:** smaltts has no TypeScript and no `typecheck` (by its M0 decision: tooling from smaltty) |
| Outside code: the rule word for word in every `CLAUDE.md` | Missing from smaltcy, smaltfl, smaltgl and smaltml |
| Session rules ("Issues") | Missing from the `CLAUDE.md` of smaltcy, smaltgl, smaltml, smaltkit, smaltnb, smaltts, smaltty and smaltui. The newest repositories (smaltcy, smaltgl, smaltml) use a different `CLAUDE.md` layout from the rest |
| Engine compatibility | Most packages require smaltdb `>=0.9.0 <0.10.0`; smaltgl requires `^0.8.3`; smaltkit allows `>=0.8.3 <0.10.0` |
| Broken reference | smaltnb's peer dependency is `smaltql`, but the package is `@smaltworks/smaltql` |

### Not yet covered by a standard

| Area | What's there |
| --- | --- |
| Node version (`engines`) | `>=20` (most), `>=22.18` (smaltkit, smaltnb, smaltty, smaltdb-sucrase), none (smaltbi, smaltui) |
| Test runner | vitest (most); `node --test` (smaltbi, smaltkit, smaltty, smaltdb-sucrase); a script of its own (smaltui) |
| Licence holder | All MIT, with three copyright lines: "Steve TenHoor" (14), "smaltdb contributors" (5), "smaltfl contributors" / "smaltql contributors". smalt-brand has no `LICENSE` |
| Default branch | `main` everywhere except **smaltdf, whose only branch is `claude/initial-design`** |
| Releases | Only smaltty has a release workflow |
| Linting and formatting | None anywhere. Consistent; possibly on purpose under the outside-code rule |

### The family

- **smaltgl** isn't in smalt-brand's family table and has no layer.
- **smalton, smaltml-simd and smalt-test** are empty: no `CLAUDE.md`, package or CI.

## Proposed changes to other repositories

Grouped by what they need from the owner. Nothing here has been done.

### A. Point every repository at smaltworks (mechanical)

In each repository's `CLAUDE.md`, add a short first section, the same way they point to smalt-brand today:

> **Direction: read smaltworks first.** Read [smaltworks CONTEXT.md](https://github.com/stenhoor/smaltworks/blob/main/CONTEXT.md)
> before starting work. Its standards, decisions and ADRs win over any copy here, unless smaltworks' EXCEPTIONS.md lists
> this repository.

One PR per repository, 19 repositories (all with a `CLAUDE.md`). Nothing else in them changes in this step.

### B. Remove the originals (the removal PRs, after this hub is merged)

- **smalt-brand:** replace `WORK.md`, ADRs 0024–0026 and 0032, the release notes and the release-notes template with
  links here; trim the direction sections out of `DECISIONS.md` and `BRAND.md`, leaving the brand parts and a link.
- **Each repository with tier 1 ADRs** (smaltdb, smaltts, smaltml, smaltty, smaltbi, smaltcy, smaltgl, smaltfl):
  replace each copied ADR with a one-line pointer here, so old links still land somewhere.
- **Every `CLAUDE.md`:** replace the copied session rules ("Working rules for sessions", "Issues") with a link to
  [context/session-rules.md](../context/session-rules.md), keeping only rules specific to the repository.

### C. Fixes that need a decision first

| # | Repository | Change | Decision needed |
| --- | --- | --- | --- |
| C1 | smaltdf | Create `main` from `claude/initial-design` and make it the default branch | Yes, it's a repository setting |
| C2 | smaltnb | Peer dependency `smaltql` → `@smaltworks/smaltql` | Or wait for the 0.9.0 renames |
| C3 | smaltgl, smaltkit | smaltdb range → `>=0.9.0 <0.10.0` | Confirm 0.9 is the floor |
| C4 | smaltdb-node, smaltdf | TypeScript 5.9 → the standard (check on 7.0, emit on 6.0.3) | — |
| C5 | smaltgl | TypeScript 7.1 beta → 7.0, or record an exception | Which |
| C6 | smaltcy, smaltml, smaltbi, smaltui | Emit `.d.ts` on 6.0.3 for one more release, or record that they don't emit `.d.ts` | Check each |
| C7 | smaltts | Record "no TypeScript; tooling from smaltty" as an exception | Confirm |
| C8 | smaltcy, smaltfl, smaltgl, smaltml | Add the outside-code rule to `CLAUDE.md` (or rely on step A's pointer) | smaltfl: does the rule bind its maintainers? |
| C9 | all | A standard Node version | `>=20` or `>=22.18` |
| C10 | all | A standard test runner, or none | vitest, `node --test`, or leave to each repository |
| C11 | all | A standard copyright line | "Steve TenHoor", "smaltworks contributors", or per repository |
| C12 | smalt-brand | Add `LICENSE` | Which licence (the site and assets may want a different one) |
| C13 | smalton, smaltml-simd, smalt-test | Scaffold to the standard, or archive | Which |
| C14 | smaltgl | Place it in a layer (direction), then in the family table (brand) | Which layer |
| C15 | smaltkit | Start ADRs for its shared components | Whether |
