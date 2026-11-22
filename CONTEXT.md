# Base context for every smalt repository

Read this first, in any smalt repository. It is short on purpose: it says what the project is for and where the rules
are. A repository's own `CLAUDE.md` adds only what is specific to that repository.

## The goal

**Small, Alternative projects supporting small data in the browser, on a unified platform, in a common language:
TypeScript.** Server work isn't ruled out; it just isn't the primary focus. (The owner, 2026-11-15.)

What follows from it, as already decided (the records are linked):

- **The browser first.** Output is plain ES2022 modules that load in a page without a bundler
  ([STANDARDS.md](STANDARDS.md#typescript-and-browsers)).
- **TypeScript everywhere.** The latest stable TypeScript, within one minor release
  ([STANDARDS.md](STANDARDS.md#typescript-and-browsers)); smaltty is smalt's own TypeScript implementation
  ([DECISIONS.md](DECISIONS.md#smaltty-typescript-without-outside-code)).
- **One platform.** smaltdb is the engine; the languages, kits and user tools are built on it. Optional features and
  all outside code live in plugins, never in core ([DECISIONS.md](DECISIONS.md#outside-code-ask-first-and-plugins-carry-it),
  [smaltdb ADR 0012](adr/imported/smaltdb/0012-plugins.md)).
- **Ask before adding outside code** ([context/session-rules.md](context/session-rules.md)).

## Where the rules are

| Need | Go to |
| --- | --- |
| The rules every repository follows | [STANDARDS.md](STANDARDS.md) |
| Where a repository departs from them | [EXCEPTIONS.md](EXCEPTIONS.md) |
| Why a rule is what it is | [DECISIONS.md](DECISIONS.md) |
| Project-wide architecture decisions, and the index of all ADRs | [adr/README.md](adr/README.md) |
| The next ADR number | [adr/README.md](adr/README.md#numbering) |
| How sessions work: PRs, issues, outside code | [context/session-rules.md](context/session-rules.md) |
| Cross-repository work in flight | [WORK.md](WORK.md) |
| What waits on the owner | [DECISION-QUEUE.md](DECISION-QUEUE.md) |
| Brand: names, voice, marks, colours, the site kit | [smalt-brand](https://github.com/stenhoor/smalt-brand) ([what goes where](docs/brand-and-works.md)) |

## The family

| Repository | Layer | What it is | npm package today |
| --- | --- | --- | --- |
| [smaltdb](https://github.com/stenhoor/smaltdb) | Engine | The engine: a database for the browser, in TypeScript | `smaltdb` |
| [smaltdb-io](https://github.com/stenhoor/smaltdb-io) | Engine plugin | CSV, TSV and JSON: reading, import and export | `@smaltdb/io` |
| [smaltdb-external](https://github.com/stenhoor/smaltdb-external) | Engine plugin | External data sources: HTTP(S), S3, Azure Blob Storage, local files | `@smaltdb/external` |
| [smaltdb-parquet](https://github.com/stenhoor/smaltdb-parquet) | Engine plugin | Parquet for external tables and `read_parquet()` | `@smaltdb/parquet` |
| [smaltdb-node](https://github.com/stenhoor/smaltdb-node) | Engine plugin | Files on disk in Node.js | `@smaltworks/smaltdb-node` |
| [smaltdb-sucrase](https://github.com/stenhoor/smaltdb-sucrase) | Engine plugin | Retired: replaced by smaltty; kept for reference | `@smaltworks/smaltdb-sucrase` |
| [smaltql](https://github.com/stenhoor/smaltql) | Languages | SQL | `@smaltworks/smaltql` |
| [smaltdf](https://github.com/stenhoor/smaltdf) | Languages | Dataframes | `@smaltworks/smaltdf` |
| [smaltts](https://github.com/stenhoor/smaltts) | Languages | The typed API | `@smaltworks/smaltts` (private) |
| [smaltcy](https://github.com/stenhoor/smaltcy) | Languages | Cypher and graphs | `@smaltworks/smaltcy` |
| [smaltkit](https://github.com/stenhoor/smaltkit) | Kits | The reusable components of the apps | `smaltkit` |
| [smaltml](https://github.com/stenhoor/smaltml) | Kits | Machine learning | `@smaltworks/smaltml` |
| [smaltml-simd](https://github.com/stenhoor/smaltml-simd) | Kits (plugin) | WebAssembly SIMD kernels for smaltml (proposed, [ADR 0035](adr/imported/smaltml/0035-simd-kernels-in-a-plugin.md)); empty | – |
| [smaltlp](https://github.com/stenhoor/smaltlp) | Kits | Linear and mixed-integer programming (proposed, [ADR 0039](adr/0039-smaltlp-scope-and-architecture.md)): `linprog()`, `milp()` and a PuLP-style modelling layer over a sparse dual simplex, compared with scipy, PuLP and HiGHS | – |
| [smaltty](https://github.com/stenhoor/smaltty) | Base | smalt's TypeScript implementation | `smaltty` (private) |
| [smaltui](https://github.com/stenhoor/smaltui) | User tools | The query workbench | `smaltui` (private) |
| [smaltbi](https://github.com/stenhoor/smaltbi) | User tools | Reports and business intelligence | `smaltbi` (private) |
| [smaltnb](https://github.com/stenhoor/smaltnb) | User tools | Notebooks | `smaltnb` |
| [smaltfl](https://github.com/stenhoor/smaltfl) | User tools | Data pipelines (a shared project with its own maintainers) | `smaltfl` (private) |
| [smaltgl](https://github.com/stenhoor/smaltgl) | Kits | Maps on the GPU | `@smaltworks/smaltgl` |
| [smalton](https://github.com/stenhoor/smalton) | Not yet placed | Ontology; empty | – |
| [smalt-test](https://github.com/stenhoor/smalt-test) | – | Empty; to be archived (2026-11-15) | – |
| smaltup | – | The web console: pick components and versions, start working in the browser; no repository yet | – |
| [smaltai](https://github.com/stenhoor/smaltai) | Alongside | The AI router and the agent in the apps ([ADR 0043](adr/0043-smaltai-router-grants-and-phases.md); [ADR 0025](adr/imported/smalt-brand/0025-smaltai-agent-gateways-and-actions.md), [0026](adr/imported/smalt-brand/0026-smaltai-in-browser-llm.md)) | – |
| [smalt-brand](https://github.com/stenhoor/smalt-brand) | – | Brand and smaltworks.dev | – |
| [smaltworks](https://github.com/stenhoor/smaltworks) | – | Direction: this repository | – |

The standard npm name is `@smaltworks/<repository>` ([STANDARDS.md](STANDARDS.md#npm-names)); the renames are part
of the 0.9.0 release ([WORK.md](WORK.md)).
