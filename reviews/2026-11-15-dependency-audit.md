# Dependency audit, 2026-11-15

Where the smalt repositories include outside code or reach outside sites. Release side first (what ships to users),
then development tools. From clones of each repository's default branch taken on 2026-11-15 (smaltdf from `main`).
Read-only: nothing was changed.

## Release side: what ships

### Outside packages in published code

Only two packages in the whole family ship outside code, both plugins, as the outside-code rule intends:

| Package | Outside code it ships | Licence | Notes |
| --- | --- | --- | --- |
| smaltdb-parquet | `hyparquet` 1.31.4 | MIT | The Parquet reader |
| | `hyparquet-compressors` 1.1.2, and through it `fzstd` 0.1.1 and **`hysnappy` 1.1.1** | MIT | **hysnappy is Snappy compiled to WebAssembly**, someone else's library. That conflicts with the WebAssembly criteria confirmed today (criterion 3: our own code only; and criterion 1, since hyparquet already decodes Snappy in JavaScript) |
| smaltdb-sucrase | `sucrase` 3.35.1 and its 7 dependencies | MIT | Retired (EXCEPTIONS.md); not to be published |

Everything else ships only family code. The engine, the languages, the kits and the apps have no outside runtime
dependencies, and no outside licence headers were found in any `src/`.

### Family dependencies that need fixing before release

| Package | Problem |
| --- | --- |
| smaltgl | Installs smaltkit from the fork `apbeers/smaltkit`, and takes smaltdb `^0.8.3` (the old unscoped release) |
| smaltkit | Takes smaltty from a git tag (`github:stenhoor/smaltty#v0.14.0`), and allows smaltdb `>=0.8.3` |
| smaltnb | Peer dependencies of `*` on six family packages (any version at all), and `smaltql` by its old name |
| smaltbi, smaltcy, smaltui | smaltkit from a git commit (`github:stenhoor/smaltkit#…`) |

All of these become published-version ranges under ADR 0037.

### Reaching outside sites at run time

| Where | What | Concern |
| --- | --- | --- |
| **smaltkit `src/engine.ts`** | The engine loader can load smaltdb from **jsDelivr** (`?engine=npm`), and does by default when an app's bundled copy is missing (`npm: true` is the default) | Loads code from another site at run time, which the outside-code rule names and the console decision replaced with fetching from npm directly. Also the beginnings of a version picker, worth folding into smaltup's approach |
| **smaltui** (`index.html`, `site/console.html`), **smaltbi** (`designer.html`), **smaltnb** (`app/index.html`), **smaltfl** (`src/app/page.html`) | The apps load the brand fonts (Geist, Geist Mono, Martian Mono) from **Google Fonts** on every visit | Not code, but every user's browser contacts Google, and the apps don't work fully offline. Self-hosting the fonts (both are open-licensed) would fix it; smalt-brand's call for the site, the owner's for the apps |
| smaltkit `src/urls.ts`, smaltui | Rewrite or suggest data URLs (GitHub raw, Hugging Face) that the **user** enters | Fine: the user's own data, not our code |

### Leftovers

- **smaltbi** still lists `hyparquet-compressors` as a development dependency, although nothing imports it: smaltbi ADR
  0006 moved it to the Parquet plugin.

## Development side: what's common and what isn't

| Tool | Repositories | Versions in use | Notes |
| --- | --- | --- | --- |
| `typescript` | 17 | 5.9, 6.0.3, 7.0, 7.0.2, 7.1.0-beta | Five versions; the standard is checking on 7.0, emitting on 6.0.3 |
| `@types/node` | 17 | `^22.0.0`, `^22.20.5` | Common |
| `esbuild` | 15 | 0.25.12, 0.28.2 | smaltnb is on 0.25 |
| `vitepress` | 13 | 1.6.4 | The docs sites; approved as a build-time tool (smaltbi ADR 0006) |
| `vitest` | 12 | 5.0.5, 5.1.0 | Recorded exception to `node --test` |
| `typescript7` (alias) | 10 | 7.0.2 | The TypeScript 7 checker beside 6.0.3 |

Used by one repository only:

| Tool | Repository | Purpose |
| --- | --- | --- |
| `@duckdb/duckdb-wasm`, `@electric-sql/pglite`, `@sqlite.org/sqlite-wasm` | smaltdb | Benchmarks and comparisons against other engines |
| `fast-check` | smaltdb | Property-based tests |
| `playwright` | smaltml | Browser tests (installed as a package; other repositories use the container's copy) |
| `tsrs`, `@tsrs/wasm`, `sucrase` | smaltty | TypeScript 7.1 early warning, and the reference its stripper is tested against |
| `hyparquet-compressors` | smaltbi | Unused (above) |

Nothing in the development tools reaches a published package.

## Suggested next steps (for the owner)

1. ~~**hysnappy:** an exception, or drop it.~~ **Decided 2026-11-15: kept for now, as a recorded exception**
   (EXCEPTIONS.md): it is in a plugin, not core, and provides a function plain JavaScript packages don't.
2. **smaltkit's jsDelivr path:** remove it, or switch it to fetching from npm directly, as decided for the console.
3. **Google Fonts in the apps:** self-host the fonts, or record an exception.
4. **The family dependencies** above move to published ranges as each package is first published (ADR 0037).
5. **smaltbi:** drop the unused `hyparquet-compressors`.
