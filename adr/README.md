# Architecture Decision Records

The index of every smalt ADR, in every repository, and the home of the project-wide ones.

## Two tiers

- **Tier 1, project-wide:** a decision that sets direction for the family, binds more than one repository, creates or
  reshapes a product or repository, or sets a rule every repository follows. Tier 1 ADRs live **here**, in `adr/`.
- **Tier 2, repository-internal:** a decision about one repository's own design that no other repository depends on.
  Tier 2 ADRs stay in their repository's `docs/adr/` and are listed in this index.

When in doubt, it is tier 1. A tier 2 ADR that turns out to bind other repositories is moved here.

The tiers below were assigned on 2026-11-15 by the rule above and accepted by the owner the same day, except for the
records under [Conflicts](#conflicts), which wait for the owner.

## Numbering

**One sequence for all smalt ADRs** (the owner's rule of 2026-11-09, first recorded in
[smalt-brand's ADR index](https://github.com/stenhoor/smalt-brand/blob/main/docs/adr/README.md)): records in every
repository share one sequence, continuing smaltdb's. A new record takes the next number after the highest in any
repository, open PRs included. Existing numbers are kept as they are; nothing is renumbered.

**The next number is 0037.**

- **Tier 1:** `adr/NNNN-short-title.md` in this repository.
- **Tier 2:** `docs/adr/NNNN-short-title.md` in its repository, with a number from the same sequence, and a row added
  here.

Known irregularities, kept as they are:

- **Repository-local sequences.** smaltbi (0001–0009), smaltcy (0001–0003), smaltgl (0001–0003) and smaltfl
  (0001–0022) number their records from 0001, so their numbers repeat numbers in the shared sequence. Refer to them
  with the repository's name: `smaltbi 0003`. smaltfl is a shared project whose maintainers decide its design in its own
  ADRs ([EXCEPTIONS.md](../EXCEPTIONS.md)); smaltbi started before the shared rule; smaltcy and smaltgl started after
  it, and take shared numbers from now on.
- **ADR 0024** is in smaltml and in smalt-brand. smaltml's copy is the newer one (amended by ADR 0034 and 0036); the
  copy here is smaltml's. smalt-brand's is out of date.

## Conflicts

Tier 1 records that conflict with another record or with a later decision, found on 2026-11-15. Each waits for the
owner; until then the later decision applies.

| # | Record | Says | Conflicts with |
| --- | --- | --- | --- |
| 1 | smaltdb 0012, 0013 | The core and its plugins release together, with the same version, in the `@smaltdb` scope; the publish plan is 0.9.0 | Independent releases, `@smaltworks` names, the 0.14.0 baseline, and the dropped 0.9.0 ([DECISIONS.md](../DECISIONS.md#releases-independent-packages-declared-compatibility)) |
| 2 | smaltdb 0015 | smaltdb, smaltql and smaltdf publish together; a smaltdb release names the language versions it was tested with; steps 11–12 publish 0.9.0 | Independent releases. Its "tested with" idea survives as the smaltworks catalog's tested sets |
| 3 | smaltfl 0017 | smaltfl builds on each family repository at its latest `main` (`deps:latest` moves every pin to the branch head) | Published versions instead of commit pins, and "dependencies come from the family" as published packages |
| 4 | smaltfl 0005 | `Copyright (c) 2026 smaltfl contributors`, and says the family uses "smaltdb contributors" | `Copyright (c) 2026 smaltworks contributors` everywhere ([STANDARDS.md](../STANDARDS.md#node-tests-branches-licence-and-dependencies)) |
| 5 | smaltcy 0001 and smaltdb 0033 (proposed) | smaltcy 0001: smaltcy has its own engine and pluggable stores. 0033: Cypher runs on smaltdb only, and smaltcy's engine goes | Each other. 0033 is proposed, so 0001 stands until 0033 is decided |
| 6 | smaltml 0035 (proposed) | smaltml-simd: WebAssembly SIMD kernels of our own, in a plugin | The goal's common language, TypeScript, and smaltty's "TypeScript only: no Rust and no WebAssembly" ([DECISIONS.md](../DECISIONS.md#smaltty-typescript-without-outside-code)). smaltml 0024 left this to the owner |
| 7 | smaltgl 0001, 0003 | smaltgl is "a product of its own"; its home is settled as `stenhoor/smalt-geo` | smaltgl is in the kits layer, and its repository is `stenhoor/smaltgl` |
| 8 | smaltbi 0008 (proposed) | Reports may load plugins from other sites at run time, pinned by hash, after the reader agrees | The outside-code rule, which names loading code from another site at run time. Needs an explicit exception or a change |
| 9 | smaltts 0023 | Its milestones re-base on, and publish with, smaltdb 0.9.0; vitest is allowed for tests | Minor: 0.9.0 is dropped (the code it names exists; the coordinated release doesn't). vitest is a recorded exception, so no conflict there |
| 10 | smaltbi 0005 | smalt-brand is consumed as an npm package of static assets | Minor, and smalt-brand's call: the site kit is copied, not packaged (review 2, section 4) |

## Imported records

Tier 1 ADRs that existed before this hub are copied, unchanged, into `adr/imported/<repository>/`. Each copy names its
source and commit at the top. The originals stay in their repositories until a later PR replaces them with links here.

## Format

New records follow [TEMPLATE.md](TEMPLATE.md), the format most existing records use.

## Index

74 records: 34 tier 1, 39 tier 2, 1 duplicate. Status as each record states it.

### The shared sequence

| No. | Repository | Title | Status | Tier | Links |
|---|---|---|---|---|---|
| 0001 | smaltdb | Initial design decisions | accepted | 1 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0001-initial-design-decisions.md) [copy](imported/smaltdb/0001-initial-design-decisions.md) |
| 0002 | smaltdb | Isolation, browser floor, packaging, results, collation, performance targets | accepted | 1 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0002-second-round-decisions.md) [copy](imported/smaltdb/0002-second-round-decisions.md) |
| 0003 | smaltdb | M1 executor model, statement journal, and catalog | accepted | 2 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0003-m1-executor-and-transactions.md) |
| 0004 | smaltdb | Write-ahead log, checkpoints, savepoints, and read-ahead | accepted | 2 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0004-wal-and-crash-safety.md) |
| 0005 | smaltdb | Cost-based optimizer (M3) | accepted | 2 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0005-cost-based-optimizer.md) |
| 0006 | smaltdb | Compact index keys (format 2) and an in-memory database mode | accepted | 2 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0006-compact-index-keys-and-in-memory-mode.md) |
| 0007 | smaltdb | Varint leaf cell headers and per-page free blocks (format 3) | accepted | 2 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0007-compact-cells-and-free-blocks.md) |
| 0008 | smaltdb | Materialized views and columnar segments (M5) | accepted | 2 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0008-materialized-views-and-columnar-segments.md) |
| 0009 | smaltdb | Concurrency (M6): request server, cancellation, streaming, multi-tab | accepted | 2 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0009-concurrency.md) |
| 0010 | smaltdb | Analytics (M7): mixed storage, spilling, columnar tables, morsels, MV matching | accepted | 2 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0010-analytics-and-mixed-storage.md) |
| 0011 | smaltdb | External tables (M8) | accepted | 2 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0011-external-tables.md) |
| 0012 | smaltdb | Plugins: a dependency-free core, optional features as packages | accepted | 1 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0012-plugins.md) [copy](imported/smaltdb/0012-plugins.md) |
| 0013 | smaltdb | Plugin API version 1 is stable | accepted | 1 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0013-plugin-api-v1.md) [copy](imported/smaltdb/0013-plugin-api-v1.md) |
| 0014 | smaltdb | A shared engine plan layer for several languages | accepted | 1 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0014-shared-plan-layer.md) [copy](imported/smaltdb/0014-shared-plan-layer.md) |
| 0015 | smaltdb | smaltql and the language API: SQL in its own repository, smaltdf on plans | accepted | 1 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0015-smaltql.md) [copy](imported/smaltdb/0015-smaltql.md) |
| 0016 | smaltdb | The plan API for every language: what smaltdf needs that SQL never asked for | accepted | 1 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0016-plan-api-for-every-language.md) [copy](imported/smaltdb/0016-plan-api-for-every-language.md) |
| 0017 | smaltdb | The remaining SQL dependencies in the engine, client and plugins | done | 1 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0017-remaining-sql-dependencies.md) [copy](imported/smaltdb/0017-remaining-sql-dependencies.md) |
| 0018 | smaltdb | `bool` and NaN | done | 2 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0018-bool-and-nan.md) |
| 0019 | smaltdb | Dates and times | accepted | 2 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0019-dates-and-times.md) |
| 0020 | smaltdb | Lists and structs | accepted | 2 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0020-lists-and-structs.md) |
| 0021 | smaltdb | Parallel readers everywhere (M9) | done | 2 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0021-parallel-readers-everywhere.md) |
| 0022 | smaltdb | SQL for the new engine features | done | 2 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0022-sql-for-the-new-engine-features.md) |
| 0023 | smaltts | smaltts, a thin typed language for smaltdb | accepted | 1 | [source](https://github.com/stenhoor/smaltts/blob/main/docs/adr/0023-the-smaltts-language.md) [copy](imported/smaltts/0023-the-smaltts-language.md) |
| 0024 | smalt-brand | smaltml's scope and architecture | accepted | duplicate | [source](https://github.com/stenhoor/smalt-brand/blob/main/docs/adr/0024-smaltml-scope-and-architecture.md) |
| 0024 | smaltml | smaltml's scope and architecture | accepted | 1 | [source](https://github.com/stenhoor/smaltml/blob/main/docs/adr/0024-smaltml-scope-and-architecture.md) [copy](imported/smaltml/0024-smaltml-scope-and-architecture.md) |
| 0025 | smalt-brand | smaltai, the AI agent in the smalt apps: gateways, actions and context | proposed | 1 | [source](https://github.com/stenhoor/smalt-brand/blob/main/docs/adr/0025-smaltai-agent-gateways-and-actions.md) [copy](imported/smalt-brand/0025-smaltai-agent-gateways-and-actions.md) |
| 0026 | smalt-brand | smaltai's in-browser LLM: an inference engine on WebGPU | proposed | 1 | [source](https://github.com/stenhoor/smalt-brand/blob/main/docs/adr/0026-smaltai-in-browser-llm.md) [copy](imported/smalt-brand/0026-smaltai-in-browser-llm.md) |
| 0027 | smaltdb | View interfaces in the engine | accepted | 1 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0027-view-interfaces.md) [copy](imported/smaltdb/0027-view-interfaces.md) |
| 0028 | smaltdb | Pluggable storage engines | deferred | 2 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0028-pluggable-storage-engines.md) |
| 0029 | smaltty | after the checker trial: speed in the page, TypeScript 7 as the reference | accepted | 1 | [source](https://github.com/stenhoor/smaltty/blob/main/docs/adr/0029-after-the-checker-trial.md) [copy](imported/smaltty/0029-after-the-checker-trial.md) |
| 0030 | smaltty | changing smaltty without breaking its consumers | accepted | 1 | [source](https://github.com/stenhoor/smaltty/blob/main/docs/adr/0030-changing-smaltty-without-breaking-its-consumers.md) [copy](imported/smaltty/0030-changing-smaltty-without-breaking-its-consumers.md) |
| 0031 | smaltdb | Processing lists and structs | accepted | 1 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0031-processing-lists-and-structs.md) [copy](imported/smaltdb/0031-processing-lists-and-structs.md) |
| 0032 | smalt-brand | git through the local repository: no credentials, no remotes | accepted | 1 | [source](https://github.com/stenhoor/smalt-brand/blob/main/docs/adr/0032-git-through-the-local-repository.md) [copy](imported/smalt-brand/0032-git-through-the-local-repository.md) |
| 0033 | smaltdb | Property graphs and version history in smaltdb | proposed | 1 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0033-property-graphs-and-history.md) [copy](imported/smaltdb/0033-property-graphs-and-history.md) |
| 0034 | smaltml | where `device: 'auto'` runs | accepted | 2 | [source](https://github.com/stenhoor/smaltml/blob/main/docs/adr/0034-where-auto-runs.md) |
| 0035 | smaltml | WebAssembly SIMD kernels, in a plugin: smaltml-simd | proposed | 1 | [source](https://github.com/stenhoor/smaltml/blob/main/docs/adr/0035-simd-kernels-in-a-plugin.md) [copy](imported/smaltml/0035-simd-kernels-in-a-plugin.md) |
| 0036 | smaltdb | User-defined functions from plugins, with predict as the first | accepted | 1 | [source](https://github.com/stenhoor/smaltdb/blob/main/docs/adr/0036-user-defined-functions.md) [copy](imported/smaltdb/0036-user-defined-functions.md) |

### Repository-local sequences (before or outside the shared rule)

Referred to with the repository's name, e.g. `smaltbi 0003`, because their numbers repeat numbers in the shared sequence.

| No. | Repository | Title | Status | Tier | Links |
|---|---|---|---|---|---|
| 0001 | smaltbi | Initial decisions | accepted | 1 | [source](https://github.com/stenhoor/smaltbi/blob/main/docs/adr/0001-initial-decisions.md) [copy](imported/smaltbi/0001-initial-decisions.md) |
| 0002 | smaltbi | Rendering with HTML, CSS and hand-written SVG | accepted | 2 | [source](https://github.com/stenhoor/smaltbi/blob/main/docs/adr/0002-rendering.md) |
| 0003 | smaltbi | A shared kit with smaltui | accepted | 1 | [source](https://github.com/stenhoor/smaltbi/blob/main/docs/adr/0003-shared-kit.md) [copy](imported/smaltbi/0003-shared-kit.md) |
| 0004 | smaltbi | Product decisions from the second interview | accepted | 2 | [source](https://github.com/stenhoor/smaltbi/blob/main/docs/adr/0004-product-decisions.md) |
| 0005 | smaltbi | smaltkit and smalt-brand repositories | accepted | 1 | [source](https://github.com/stenhoor/smaltbi/blob/main/docs/adr/0005-smaltkit-and-smalt-brand.md) [copy](imported/smaltbi/0005-smaltkit-and-smalt-brand.md) |
| 0006 | smaltbi | Outside code, smaltdf, fixed page sizes and the calendar | accepted | 1 | [source](https://github.com/stenhoor/smaltbi/blob/main/docs/adr/0006-outside-code-smaltdf-pages-calendar.md) [copy](imported/smaltbi/0006-outside-code-smaltdf-pages-calendar.md) |
| 0007 | smaltbi | Phones see every page as a responsive web page | accepted | 2 | [source](https://github.com/stenhoor/smaltbi/blob/main/docs/adr/0007-phones-responsive.md) |
| 0008 | smaltbi | Loading plugins | proposed | 1 | [source](https://github.com/stenhoor/smaltbi/blob/main/docs/adr/0008-plugin-loading.md) [copy](imported/smaltbi/0008-plugin-loading.md) |
| 0009 | smaltbi | Many-to-many | accepted | 2 | [source](https://github.com/stenhoor/smaltbi/blob/main/docs/adr/0009-many-to-many.md) |
| 0001 | smaltcy | smaltcy: a graph engine, Cypher on plans, pluggable stores and a GPU renderer | accepted | 1 | [source](https://github.com/stenhoor/smaltcy/blob/main/docs/adr/0001-smaltcy.md) [copy](imported/smaltcy/0001-smaltcy.md) |
| 0002 | smaltcy | Graph history: versions, time travel and temporal Cypher | accepted | 2 | [source](https://github.com/stenhoor/smaltcy/blob/main/docs/adr/0002-graph-history.md) |
| 0003 | smaltcy | Drawing lives in smaltkit | accepted | 1 | [source](https://github.com/stenhoor/smaltcy/blob/main/docs/adr/0003-drawing-in-smaltkit.md) [copy](imported/smaltcy/0003-drawing-in-smaltkit.md) |
| 0001 | smaltfl | Initial decisions | accepted | 1 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0001-initial-decisions.md) [copy](imported/smaltfl/0001-initial-decisions.md) |
| 0002 | smaltfl | Identity, districts, snapshots, the builder, the colour | accepted | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0002-second-round-decisions.md) |
| 0003 | smaltfl | The ontology screens, the scenario's actions, the mark on hold | accepted | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0003-third-round-decisions.md) |
| 0004 | smaltfl | The flow plan: people say what, the machine decides how | accepted | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0004-the-flow-plan.md) |
| 0005 | smaltfl | The licence | accepted | 1 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0005-licence.md) [copy](imported/smaltfl/0005-licence.md) |
| 0006 | smaltfl | Integrity from source bytes to the record | accepted | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0006-integrity.md) |
| 0007 | smaltfl | The repository moves to stenhoor/smaltfl | accepted | 1 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0007-repository-home.md) [copy](imported/smaltfl/0007-repository-home.md) |
| 0008 | smaltfl | The day's news, drawn | accepted | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0008-visual-first-news.md) |
| 0009 | smaltfl | A day that is off is paused, not published | accepted | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0009-paused-not-published.md) |
| 0010 | smaltfl | The news, the story and the pause, for any dataset | proposed | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0010-any-dataset.md) |
| 0011 | smaltfl | Notebooks as the step surface | proposed | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0011-notebooks-as-the-step-surface.md) |
| 0012 | smaltfl | Visual first, in two minutes: the brief, the incident page, the drill and the phone canvas | proposed | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0012-visual-first-in-two-minutes.md) |
| 0013 | smaltfl | Checks nobody wrote: combinations judged from the history, and a dataset the flow builds watched like a source | proposed | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0013-discovered-checks.md) |
| 0014 | smaltfl | Gates: bronze, silver and gold checks in the notebook, written bands and due dates in the ontology, a sealed gold | accepted | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0014-gates.md) |
| 0015 | smaltfl | The bronze layer: checks driven by the ontology, a mode a column, the card read as a notebook | proposed | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0015-the-bronze-layer.md) |
| 0016 | smaltfl | The mark is the Twin, in the user-tool layer's cerulean | accepted | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0016-the-mark.md) |
| 0017 | smaltfl | Built on stenhoor's repositories, at their latest main | accepted | 1 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0017-stenhoors-repositories-at-main.md) [copy](imported/smaltfl/0017-stenhoors-repositories-at-main.md) |
| 0018 | smaltfl | Phone, tablet and desktop | accepted | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0018-phone-tablet-and-desktop.md) |
| 0019 | smaltfl | Ranges computed ahead, and a first load measured | accepted | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0019-ranges-computed-ahead.md) |
| 0020 | smaltfl | The estate as a company: an organisation, shared infrastructure, and a real bronze, silver and gold | accepted | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0020-the-estate-as-a-company.md) |
| 0021 | smaltfl | Time: a month to pick a day from, the day itself, and every version of it | accepted | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0021-time.md) |
| 0022 | smaltfl | The answers of 2026-11-14: a what-if and a restatement decided where they are, columns that change, the ontology as the one place for rules, and pausing | accepted | 2 | [source](https://github.com/stenhoor/smaltfl/blob/main/docs/adr/0022-the-answers-of-2026-11-14.md) |
| 0001 | smaltgl | smaltgl | accepted | 1 | [source](https://github.com/stenhoor/smaltgl/blob/main/docs/adr/0001-smaltgl.md) [copy](imported/smaltgl/0001-smaltgl.md) |
| 0002 | smaltgl | Map visuals for hosts | accepted | 1 | [source](https://github.com/stenhoor/smaltgl/blob/main/docs/adr/0002-visuals-for-hosts.md) [copy](imported/smaltgl/0002-visuals-for-hosts.md) |
| 0003 | smaltgl | The visualizations live in smaltkit | accepted | 1 | [source](https://github.com/stenhoor/smaltgl/blob/main/docs/adr/0003-visuals-in-smaltkit.md) [copy](imported/smaltgl/0003-visuals-in-smaltkit.md) |
