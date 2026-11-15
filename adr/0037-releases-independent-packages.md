# ADR 0037: Releases: independent packages, declared compatibility

- **Status:** accepted (2026-11-15, the owner, in interviews the same day)
- **Date:** 2026-11-15
- **Tier:** 1 (project-wide)
- **Repositories:** every repository that publishes to npm
- **Supersedes:** the release clauses of smaltdb ADR [0012](imported/smaltdb/0012-plugins.md) ("Releases", "Before the
  first publish", step 7), [0013](imported/smaltdb/0013-plugin-api-v1.md) (core and plugins released together) and
  [0015](imported/smaltdb/0015-smaltql.md) (smaltdb, smaltql and smaltdf released together, steps 11–12);
  smaltfl ADR [0017](imported/smaltfl/0017-stenhoors-repositories-at-main.md) (pins moved to each repository's latest
  `main`); the 0.9.0 milestones of smaltts ADR [0023](imported/smaltts/0023-the-smaltts-language.md). The rest of those
  records stands.

## Context

The family had planned one coordinated release, 0.9.0, of smaltdb, its plugins and the languages, together. It was in
progress from 2026-11-03 and never went out. Until it did, every repository installed the others from commits pinned in
`packed.json`, and each pinned its own: seven different smaltdb commits and seven different smaltkit commits were in use
across the family (review 2, section 1). Lining up this many repositories for one release was judged too fragile.

## Decision

1. **The coordinated 0.9.0 release is dropped.** Packages release one at a time from their own repositories.
2. **Compatibility is declared, not implied.** `peerDependencies` ranges say what a package works with. Version numbers
   never indicate compatibility.
3. **Contracts have their own versions:** the plugin API (v1), the plan format (v0, experimental), the language API
   (v1) and the storage format (v3). Before 1.0, a breaking change bumps the package's minor version and the version of
   any contract it changes.
4. **Two npm channels, both on demand:** `dev` (prereleases numbered in sequence; consumers pin them exactly) and
   `latest` (stable; consumers use ranges). Nothing publishes automatically.
5. **Names and starting versions:** every package is `@smaltworks/<repository>` and starts at 0.14.0. The unscoped
   `smaltdb` 0.8.3 is deprecated with a pointer once `@smaltworks/smaltdb` is out.
6. **Each package describes itself** in a `"smalt"` block in `package.json` (layer, environment, contracts provided and
   required, how it registers); smaltworks generates a catalog from what is published, including the sets tested
   together.
7. **Publishing** is one workflow of our own, a template in smaltworks copied to each repository, using npm trusted
   publishing. Collaborators may publish dev builds of any package; stable releases only by the package's owner.
8. **Release notes** are one consolidated set in each GitHub release, Breaking first, drawn from the merged PRs.
9. **First dev publishes go from the engine outward,** and consumers then replace their `packed.json` pins with
   published versions. `scripts/fetch-packed.mjs` and `packed.json` go once nothing needs them.

## Consequences

- Each repository moves when it is ready; nobody waits for the slowest.
- The catalog becomes the place to see what goes with what; smaltdb 0015's "a release names the versions it was tested
  with" lives on there as tested sets.
- smaltfl stops following other repositories' latest `main` and depends on published versions, like every other
  consumer.

## Considered

- **Lockstep** (one version, released together): the approach being superseded.
- **Publishing on every merge:** rejected for on-demand publishing.
- **Changesets and release-please:** outside code, and both still need custom steps for contracts and the catalog.
- **One repository for the engine group:** rejected; separate repositories enforce the divisions of labour and code.
