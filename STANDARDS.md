# Project standards

The rules every smalt repository follows. Where a repository departs from one, the exception is recorded in [EXCEPTIONS.md](EXCEPTIONS.md). The decisions behind them are in [DECISIONS.md](DECISIONS.md).

> **Copied** from [smalt-brand/BRAND.md](https://github.com/stenhoor/smalt-brand/blob/9c78b5f363185190c70d88b24deee22e6fc78f5f/BRAND.md) (commit `9c78b5f`) on 2026-11-15: the sections about project direction, unchanged, in their original order. The brand sections stay in smalt-brand. The source stays in place until its removal PR.

## npm names

Packages publish to npm under the smaltworks organisation, named after their repositories: `@smaltworks/smaltdb`, `@smaltworks/smaltql`, `@smaltworks/smaltdf`, `@smaltworks/smaltdb-parquet` and so on. The repository name is the package name, so there is one name to remember for each project. smaltdb 0.8.3, the last release under the unscoped name `smaltdb`, stays on npm.

Ready-made distributions (the engine alone, or the engine with particular languages) are planned but not yet named.

## TypeScript and browsers

Family rules for every smalt repository (owner, 2026-11-12, from [smaltty ADR 0030](https://github.com/stenhoor/smaltty/blob/main/docs/adr/0030-changing-smaltty-without-breaking-its-consumers.md), decisions 5 and 6). The details are in DECISIONS.md, "TypeScript and browsers".

- **TypeScript: the latest stable, within one minor release.** Type checking (`typecheck`) runs on TypeScript 7 (7.0.2). Whatever emits `dist/` or `.d.ts` stays on TypeScript 6.0.3 for one more release, while 7's output is compared with it, then moves to 7. smaltty keeps 6.0.3 for its compiler API until TypeScript 7 has a stable JavaScript API (7.1 at the earliest). tsrs, a Rust port of TypeScript 7's checker that follows its next version, is the early warning.
- **Browsers: plain ES2022 modules,** which load in a page without a bundler. smaltty's built-in types follow web-features Baseline "widely available". The target and the baseline are reviewed once a year, in a decision record; the next step is Baseline 2024.

## Documentation

Every product's documentation follows one standard, for people and for the AI agents that use the products. The
mock-ups are in the [exploration archive](https://github.com/stenhoor/smalt-brand/blob/main/exploration/README.md#documentation-standard) and on the family canvas; the
decisions and what was considered are in [DECISIONS.md](DECISIONS.md#documentation-standard). The components marked
*to build* are not in the site kit yet: until they are, a site follows the rules it can, starting with smaltql.

1. **Four sections:** Quick start, Getting started, User guide and Reference, in that order and with those names. The
   standard does not change a product's home page. The top nav is Quick start, Guide, Reference, Playground, then the
   smaltworks menu (an app site puts Demo first: Sites and READMEs, rule 12). A site leaves a section out only where
   this table does:

   | Site | Quick start | Getting started | User guide | Reference |
   |---|---|---|---|---|
   | smaltdb | yes | yes | Engine concepts | API, the plan language, errors |
   | smaltql, smaltdf | yes | yes | Tasks in the language | The language (statements or methods, functions), its API |
   | smaltts | yes | yes | Specs, changes, flows | API |
   | smaltcy | yes | yes | Tasks in the language | The language (clauses, functions), its API |
   | smaltui, smaltbi, smaltnb, smaltfl | yes | yes | Working in the app | Formats (report, notebook, flow), shortcuts |
   | Plugins | yes | no | Per plugin | Functions, options, API |
   | smaltml | yes | yes | Tasks: preparing data, training, evaluating, saving, predicting | API, one estimator or function per page |
   | Kits (smaltkit, smaltty) | yes | no | no | API, module by module |

2. **Quick start:** one page, five minutes, three steps at most, ending in a live playground, so a reader who installs
   nothing still runs something real.
3. **Reference entries:** one shape for every function, method, statement and plan operation: the name; the signature
   with its types; a one-line summary; parameters; what it returns (or its output columns); the errors it raises, with
   their codes; notes; examples with their printed results; a working demo; see also. **One function, method or
   statement per page**, grouped in the sidebar under its kind (String, Math, Aggregate …); a group's page lists its
   members and states the rules they share. Names for the same function (`substr`, `substring`) share a page. A
   language's functions and statements have syntax diagrams, as its grammar does (rule 4).
4. **Syntax diagrams:** every statement and grammar rule of a language has a railroad diagram, drawn from its grammar
   (EBNF, in the W3C notation) by the site kit's generator, our own code (`railroad.ts`): keywords in rounded boxes,
   rules in square boxes that expand in place, long rules wrapped onto rows, and the EBNF beside them. A site keeps its
   grammar in one file, which the diagrams and the agent guide both read. Prose after the diagrams says what they
   can't, such as where an aggregate may appear.
5. **Examples are tests:** every example runs in CI, and its printed result is checked against what it gives.
6. **Working demos:** every language and API reference page ends with a complete small program (its data, its code
   and its result) that runs as written.
7. **Playgrounds:** on every guide and reference page, one component from the site kit (`playground.ts`), built from
   smaltkit (`editor` and `grid`). **Every tested example runs in place:** at rest it is the example, a small Run
   button and its tested result (or the error it is tested to raise), and it loads nothing; Run or Edit turns it into
   a playground where it stands. A fence marked `noplay` doesn't run, nor does an example that needs a plugin the site
   doesn't load; the first Run starts the engine, from the site's own copy and never a CDN, in a
   worker, once per visit. Each Run starts from the page's setup: the tested examples above it, as the examples test
   runs them, in an in-memory database. Run, Edit, Reset, Copy and Share (the code travels in the link's fragment,
   never to a server), and a link to the app where the app can open code from one. What a playground holds, and what it
   leaves to the app, is in Sites and READMEs, rule 12.
8. **The plan format is a language:** smaltdb's reference has a Plan language section, like a language's: syntax
   diagrams of the JSON, an entry for each operation and expression kind, and playgrounds that run plan JSON, with a
   tree view for reading a plan. It opens with "One query, every language": the same query in SQL and in smaltdf, and
   the plan both build. Every SQL and smaltdf playground has a Plan tab (*to build*: smaltql's plan of a statement and
   the plan tree printer). The section is marked experimental, beside the work-in-progress badge, until the plan
   format's stability ADR.
9. **Docs for agents:** every site also publishes Markdown for AI agents, generated from the same sources as its pages
   (the site kit's `agents.ts`, at the end of the build): a Markdown twin of every page (`page.md`), an `/llms.txt`
   index (the [llms.txt proposal](https://llmstxt.org/)), and one agent guide per product (`/agents.md`). An agent guide
   is complete rather than friendly: how to use the product, its rules (a page the site writes for agents), its grammar
   as EBNF text, every signature, tested examples with their results, every error code, and what isn't supported, with
   the alternative (the reference pages it names, in the sidebar's order). Each file names the
   package's version (and the plan format's, where plans appear). The same generator feeds smaltai's context documents
   ([ADR 0025](adr/imported/smalt-brand/0025-smaltai-agent-gateways-and-actions.md)).
10. **Voice:** this file's, as everywhere: what it is, short sentences, the stage badge, no number without its
    benchmark.

## Plugins

A plugin gets a product brand of its own only when people choose it and use it directly, as with smaltdf (`@smaltworks/smaltdf`). Infrastructure plugins (`@smaltworks/smaltdb-parquet`, `@smaltworks/smaltdb-io`, `@smaltworks/smaltdb-external`) use smaltdb's brand and a plain name, such as "smaltdb Parquet", with no mark or colour of their own.

Three badges, in `badges/plugins/`, say how a plugin relates to smaltdb:

| Tier | Meaning | Badge |
| --- | --- | --- |
| **Official** | Made and maintained by smaltworks, in the `@smaltworks` npm scope | The product's mark, the wordmark, "official plugin" |
| **Verified** | Made by someone else; passes the smaltworks compatibility tests for a stated plugin API version | A check, "Verified for", the wordmark, an "API v1" tag |
| **Works with** | The author says it works; smaltworks hasn't checked | "Works with" and the wordmark in grey, with a dashed border and no mark |

- Product marks appear only on what smaltworks makes. Brand colours appear only on what smaltworks makes or has tested.
- Official and Verified badges link to the plugin's entry on smaltdb.dev/plugins, the list smaltworks controls. A badge image alone proves nothing, since anyone can copy it.
- Verified means the compatibility tests pass for that API version. It is not a security review, and the list page says so.
- The `@smaltworks` npm scope and names that start with `smalt` are reserved for smaltworks. Third-party plugins are named `<product>-plugin-<name>`, for example `smaltdb-plugin-geo`.

## Default branch

Every smalt repository's default branch is `main`, and work branches merge into it. A session's working branch
(`claude/…`) is never a default branch. (DECISIONS.md, "Default branch: main", 2026-11-15.)

## Versions and releases

- Each package versions and releases on its own, from its own repository, on demand: `dev` for prereleases, `latest`
  for stable. Packages under `@smaltworks` start at 0.14.0.
- **Version numbers never indicate compatibility.** `peerDependencies` ranges and contract versions do.
- Before 1.0, a breaking change bumps the minor version, and the version of any shared contract it changes.
- Release notes are written as one consolidated set in the release, Breaking first, drawn from the merged PRs.
- (DECISIONS.md, "Releases: independent packages, declared compatibility".)

## Node, tests, branches, licence and dependencies

- **Node `>=22.18`** in every package's `engines`, and in CI.
- **Tests run with `node --test`.** Browser tests are our own scripts. Repositories on vitest are listed in
  [EXCEPTIONS.md](EXCEPTIONS.md).
- **"Automatically delete head branches"** is on in every repository.
- **Every `LICENSE` is MIT, `Copyright (c) 2026 smaltworks contributors`.**
- **Dependencies come from the family:** `stenhoor` repositories or published `@smaltworks` packages only, unless
  [EXCEPTIONS.md](EXCEPTIONS.md) lists the case.
- (DECISIONS.md, "Standards from the review interviews", 2026-11-15.)

## WebAssembly

No WebAssembly in the core. A plugin may use it only in the circumstances set out in DECISIONS.md, "WebAssembly:
plugins only" (drafted 2026-11-15, waiting for the owner's confirmation).
