# Direction decisions

What the owner decided about the project's direction, when and why. A newer decision wins over an older copy anywhere else. New decisions are added here, dated, with their reasons.

> **Copied** from [smalt-brand/DECISIONS.md](https://github.com/stenhoor/smalt-brand/blob/9c78b5f363185190c70d88b24deee22e6fc78f5f/DECISIONS.md) (commit `9c78b5f`) on 2026-11-15: the sections about project direction, unchanged, in their original order. The brand sections stay in smalt-brand. The source stays in place until its removal PR.

## Plugins and smaltkit

Reasoned through in conversation, then mocked up on one board.

- **smaltkit gets no brand:** it is shared code inside the apps, not a product people choose. It gets only a README header in the wordmark style. (Changed 2026-11-14: smaltkit has a brand, "smaltkit: a brand" above.)
- **Product brands for plugins:** only when people use the plugin directly (smaltdf); infrastructure plugins use smaltdb's brand.
- **Three tiers:**
  - **Official:** from smaltworks, in the `@smaltdb` scope.
  - **Verified:** smaltworks-approved, after passing compatibility tests.
  - **Works with:** self-attested by the author.
- **On the words:**
  - "Compatible" was proposed for the approved tier; "Verified" was chosen because readers can't tell "Compatible" from "Works with".
  - "Reviewed" and "Certified" were ruled out because they imply a security or formal review.
- **Badges:** Official carries the product's own mark, so an official smaltnb plugin would show the Pencil, not the Grain. Verified swaps the mark for a check and adds the API version. Works with has no mark, no brand colour and a dashed border.
- **Proof:** Official and Verified badges link to a list on smaltdb.dev/plugins that smaltworks controls.

All approved by the owner. Still to build before Verified can be granted: the compatibility test suite and the smaltdb.dev/plugins page.

## Outside code: ask first, and plugins carry it

Decided by the owner on 2026-11-08, after a session added third-party tools (Prettier, then sucrase) to an app without
asking, and then took them out again without discussing it.

- **Ask before adding outside code.** A session asks the owner, and waits for a yes, before it adds a package (a
  dependency of any kind, or a pin in `packed.json`), copies or bundles in someone else's code, or loads code from
  another site at run time (a CDN such as jsDelivr). It says what the code is, its licence and size, what it's for, and
  why our own code can't do it. Every repository's `CLAUDE.md` carries the rule, word for word, under "Working rules for
  sessions".
- **Plugins carry outside code.** Outside code that a feature needs goes in a plugin, so it is named and optional and
  its licence travels with it, never in core. This extends smaltdb ADR 0012 from the engine to the whole family. The
  first cases:
  - **Formatting:** JSON and TypeScript formatting is our own code (`smaltkit/pretty`), not Prettier.
  - **Compiling TypeScript at run time:** a plugin, `@smaltworks/smaltdb-sucrase`, replaces loading the TypeScript
    compiler from jsDelivr in smaltui's TypeScript tabs. (Later the same day, our own code replaced it: see "smaltty:
    TypeScript without outside code".)
  - **Parquet codecs:** the Parquet plugin owns `hyparquet-compressors` as its own dependency, instead of each app
    installing it.

## smaltty: TypeScript without outside code

Decided by the owner on 2026-11-08, in an interview with a session that proposed it. **Partly superseded on
2026-11-12:** smaltty is now a top-level product ("smaltty: a product", below). The rest of this section stands.

- **Why:** smaltui stripped TypeScript types with sucrase, and smaltnb loaded the TypeScript compiler (8.7 MB) to run
  notebook cells. Both brought outside code into the apps, and neither checked types, so code written against smaltdf
  lost its type checking just before it ran.
- **What:** [smaltty](https://github.com/stenhoor/smaltty), our own TypeScript kit: a parser, a stripper, and later
  scope analysis and type checking against smaltdf. In order: remove the outside code first, measure, and only then
  decide how far checking goes.
- **A kit, not a product** (superseded 2026-11-12, below). smaltty is one of the kits, with no brand entry, mark or site of its own. It becomes a
  top-level product only if it grows into our own TypeScript compiler, or a significant part of one, and that is the
  owner's call. smaltworks isn't in the business of writing TypeScript compilers: work that doesn't remove outside code
  or make the apps faster waits.
- **TypeScript only:** no Rust and no WebAssembly. **Follow TypeScript, not sucrase:** where they differ, TypeScript's
  behaviour is right. Types are removed, never compiled (TypeScript's `erasableSyntaxOnly`).
- **smaltkit wraps it:** apps import `smaltkit/typescript`; smaltkit carries smaltty in its build.
- **Retired:** `@smaltworks/smaltdb-sucrase`, once smaltui moved to smaltty (stenhoor/smaltui#33). Its repository stays
  for reference.
- **Later:** checking table and column names against a database is a separate future project, smaltsc (smalt schema
  checker), not part of smaltty.

## TypeScript and browsers

Family rules, decided by the owner on 2026-11-12 in smaltty's [ADR 0030](https://github.com/stenhoor/smaltty/blob/main/docs/adr/0030-changing-smaltty-without-breaking-its-consumers.md)
(decisions 5 and 6, accepted). The owner chose to make them rules for every smalt repository, recorded here as well.
BRAND.md, "TypeScript and browsers", has the rules; smaltworks.dev's supported environments page says what they mean
for users.

- **TypeScript: the latest stable, within one minor release,** in every smalt repository. Now, in two steps:
  - **Checking moves to TypeScript 7 (7.0.2)** in every repository: the `typecheck` scripts. It is faster, and the
    builds are compatible. The work in each repository is that repository's own session's.
  - **Emitting stays on TypeScript 6.0.3 for one more release.** Anything that builds `dist/` or `.d.ts` is built
    with 6.0.3 while 7's output is compared with it; then it moves to 7. Emitted `.d.ts` is what other repositories
    read, so it moves only once compared.
  - **smaltty keeps 6.0.3 for its compiler API** (its differential tests) until TypeScript 7 has a stable JavaScript
    API, 7.1 at the earliest.
  - **tsrs is the early warning.** tsrs, a Rust port of TypeScript 7's checker that follows its next version (7.1
    beta now), shows what the next release changes before it ships.
- **Browsers: a stated baseline.**
  - Output stays plain ES2022 modules, which every current browser loads without a bundler.
  - smaltty's built-in types follow web-features Baseline "widely available": what users can call in the browsers
    the apps support.
  - The output target and the baseline are reviewed once a year, in a decision record. The next step is Baseline
    2024 (`Array.fromAsync`, ES2025's `Set` methods), with smaltty's milestone 14.
- **Considered** (ADR 0030): moving checking and emitting to TypeScript 7 at once, and staying on an older
  TypeScript until 7's API is stable.

## Documentation standard

One standard for every product's documentation (BRAND.md, "Documentation"), from a review of three projects whose docs
the owner admires: Polars for its function entries with sample code, NumPy for its structure, SQLite for its syntax
diagrams. Ten boards were mocked up (docs round 1, in the [exploration archive](https://github.com/stenhoor/smalt-brand/blob/main/exploration/README.md#documentation-standard)).

The owner's decisions, 2026-11-10:

- **Four sections on every site:** Quick start (zero to working), Getting started, User guide, Reference.
- **Every reference entry the same shape,** with examples tested against their printed results, and a **working
  demo** on every language and API page: a complete small program a reader sees run, then changes. **Playgrounds on
  every page,** built from smaltkit.
- **Syntax diagrams from our own generator.** Mermaid has railroad diagrams (beta since v11.16), but it is outside
  code; a small generator of our own draws them in the brand's colours, and the mock-ups use it.
- **Demos and playgrounds** (Sites and READMEs, rule 12, rewritten). In the owner's words: demos are full app
  environments, the full interface; playgrounds have a stripped-down interface, fully functional, to show code actually
  running (tell and show). Playgrounds are in the documentation; apps are in Demo. Keeping playgrounds off the app
  sites, and dropping the rule's "nothing else opens the app", were considered.
- **The standard in BRAND.md, the components in the site kit** (`themes/vitepress/`). A DOCS.md of its own, and the
  components in smaltkit, were considered.
- **smaltql first:** it already has a quick start, a guide and tested examples.
- **The plan format documented as a language** (the owner's lean): the neutral way into the engine gets syntax
  diagrams, an entry per operation and expression, and plan playgrounds, opened by "One query, every language" (SQL,
  smaltdf and the plan they build). Documenting only the TypeScript builders, and only the side-by-side view, were
  considered. It needs smaltql to give the plan of a statement, which it doesn't export yet, and a printer for whole
  plans; the format stays marked experimental until its stability ADR.
- **Docs for agents** (Jake Rastberger's input): beside the people docs, Markdown made for the AI agents that use the
  languages and tools: a twin of every page, an `/llms.txt` index, and one agent guide per product, generated from the
  same tested sources.

Building the playground (stenhoor/smalt-brand#64), the owner agreed, 2026-11-10:

- **smaltkit in the docs sites.** A site with playgrounds pins smaltkit to a commit (`github:stenhoor/smaltkit#…`), as
  smaltui and smaltbi do: playgrounds are built from it, and it is our own code.
- **"Open in the app" waits for the app.** The link shows only where the site can give one that opens the code; smaltui
  can't open code from a link yet, and teaching it to is a task of its own.

Reviewing smaltql's site under the standard, the owner decided, 2026-11-10:

- **A function per page.** Function references give each function a page of its own, grouped in the sidebar under its
  kind (scalar, string, math …), in place of one page per kind with every function on it.
- **Every example runs in place.** One playground per page left the other examples on it unrunnable. Of the
  alternatives offered (every example a full playground; examples loading one shared playground; a playground per
  section), the owner chose examples that look like examples at rest, with a Run button that turns each into a
  playground where it stands.
- **SQL is a language too.** The plan format's treatment (BRAND.md rule 8) is the model for SQL's reference: syntax
  diagrams for statements and for functions, and an entry for each. The plan language itself follows, as decided.

The owner decided, 2026-11-11:

- **The home page is not part of the standard.** Rule 1 had said "the home page opens on the four as cards". The
  session that wrote the standard into BRAND.md added that line; the owner never decided it, and smaltql's home page
  was changed under it without being asked for (its six feature cards replaced by four docs cards). The line is
  removed: the standard does not change a product's home page, and smaltql's goes back to what it was
  (stenhoor/smaltql#64).

Bringing smaltcy's site to the standard, the owner chose, 2026-11-14, to extend the kit's playground rather than keep
smaltcy's own runner or drop its graphs. smaltcy's examples each read a sample graph and draw their result as a graph:

- **Attributes:** a fence's `key=value` flags (```` ```cypher dataset=metro ````), over the page's front matter
  `play:` map, reach the runner with the code and with each setup step.
- **A result can draw more** under its table (`render`), freed when the result goes.

Keeping smaltcy's own component (an exception to rule 7), and the kit as it was (datasets through `docs:setup`
markers, and grids only), were considered.

## Work items: one register

Decided by the owner on 2026-11-09, when smaltdb ADR 0022 (SQL for the new engine features) needed tracking and review
with other work, and smalt-brand had no place for it.

- **[WORK.md](WORK.md)** lists cross-repository work that is proposed, waiting for decisions or under way: one row per
  item, with its record's link, status, next step and what it waits on. A multi-step item can list its milestones
  below the table.
- **Records stay home.** An ADR or plan lives in its own repository; WORK.md links it and never copies its text, so
  there is one copy to review. Copying the ADR into smalt-brand, and a tracking issue, were considered.
- **Sessions keep it current.** A session that opens, advances or finishes such work updates its row in a smalt-brand
  PR, as product sessions already do for the status page.

## Default branch: main

Decided by the owner on 2026-11-15, when the drift review found smaltdf's default branch was still
`claude/initial-design`, a session's working branch, while its work had moved on to `main`: a PR merged into the old
default (stenhoor/smaltdf#24, the outside-code rule) never reached `main`, and anyone opening the repository saw a
five-day-old copy.

- **Every smalt repository's default branch is `main`.** Work branches merge into it.
- **Session branches are never a default branch.** A new repository starts on `main`.
- A repository's default branch is a GitHub setting (Settings → General → Default branch); a session can create
  `main` but can't change the setting.

## Releases: independent packages, declared compatibility

Decided by the owner on 2026-11-15, in an interview, after the coordinated 0.9.0 release had been in progress since
2026-11-03 without going out. Lining up every repository for one release was judged too fragile for this many
repositories.

- **The coordinated 0.9.0 release is dropped.** It never went out. Packages release one at a time from here.
- **Independent versions, declared compatibility.** Each package publishes from its own repository when it is ready.
  `peerDependencies` ranges say what it works with.
- **Contract versions.** The shared interfaces (the plugin API, the plan format) have versions of their own. Before
  1.0, a breaking change bumps the package's minor version (0.9.x → 0.10.0) and, if it changes a shared interface,
  that contract's version too. Consumers move when they are ready.
- **Two channels on npm:** `dev` (prereleases, numbered in sequence) and `latest` (stable). Both are published **on
  demand**, never automatically. Consumers pin dev builds exactly and use ranges for stable releases.
- **The consumers are the family's own apps** until the interfaces are fixed at 1.0.0. Whether outside projects start
  using the packages is checked on npm during on-demand reviews, not by standing monitoring.
- **Package metadata in both places:** a `"smalt"` block in each `package.json` (layer, browser or Node, contracts
  provided and required, how it registers) is the source; smaltworks generates a catalog from what is published.
- **Publishing:** one workflow of our own, kept as a template in smaltworks and copied to each repository: no outside
  code. Considered: Changesets and release-please (both outside code; both need custom steps for contracts and the
  catalog anyway; release-please pushes towards continuous releases).
- **Who publishes:** collaborators may publish **dev** builds of any package; a **stable** release is published only by
  the package's owner. npm trusted publishing ties each package to its repository's workflow, so no tokens are shared.
- **Where it shows:** npm holds the packages (what exists); each publish writes a GitHub release (what changed) and
  regenerates the smaltworks catalog (the family view, and what is tested together).
- **Release notes are consolidated.** They are drawn from the PRs merged since the last release, but written as one
  set of notes in the release itself, Breaking first. Readers are not sent to a list of PRs or other documents to piece
  it together.
- **Separate repositories stay.** A monorepo for the engine group was considered and rejected: separate repositories
  enforce the divisions of labour and code.

### Follow-ups, 2026-11-15

- **Starting versions:** every package under `@smaltworks` starts at **0.14.0**, the furthest-along version (smaltty's),
  as a one-time common baseline for the move to the new scope. From then on each package versions on its own.
  **Version numbers never indicate compatibility**; contract versions and peer ranges do. Considered: each package
  keeping its own number.
- **The old `smaltdb` on npm** (0.8.3, unscoped) is deprecated with a pointer to `@smaltworks/smaltdb` once that is
  published. Nothing is removed.
- **Contracts at the start:** the plugin API (v1, smaltdb ADR 0013), the plan format (v0, experimental), the language
  API, and the storage format.
- **First dev publishes go from the engine outward:** smaltdb, then its plugins and the languages, then the kits; the
  apps then replace their `packed.json` pins with published versions.

## Web console: first scope (provisional)

Decided by the owner on 2026-11-15 as "for now" answers, expected to grow; none is a firm boundary.

- **The console is a browser page where people pick smalt components and versions and start working.** It loads
  packages from npm (through a CDN) using the release channels above, checks the chosen set against peer ranges and
  contracts, and pins it as a shareable profile.
- **For now, for data users:** people who want to load data and query it in the browser. App builders, contributors
  and internal use are likely later.
- **Runs libraries, and launches the apps:** the engine, plugins and languages in the console itself, and the apps
  (smaltui, smaltbi, smaltnb) opened against the chosen versions.
- **Browser first.** Node is likely to follow at some point; not decided, no date.
