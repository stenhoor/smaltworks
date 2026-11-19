# ADR 0039: smaltlp's scope and architecture

- **Status:** proposed. The owner's answers of 2026-11-17 (an interview after the research below) are recorded under
  Decision. Amended 2026-11-18 (the owner): sections 7 and 8, WebGPU and SIMD run on smaltkit's compute, and smaltlp
  writes the kernels (DECISIONS.md, "WebGPU and SIMD in smaltkit"). Amended 2026-11-19 (the owner): the items that
  were "Still to decide" are decided, under "Decided 2026-11-19".
- **Date:** 2026-11-17
- **Tier:** 1 (project-wide): it creates a product and a repository.
- **Repositories:** smaltlp (new); smalt-brand (its brand); smaltdb (a plugin function later, through ADR 0036);
  smaltworks
- **Builds on:** smaltml [ADR 0024](https://github.com/stenhoor/smaltml/blob/main/docs/adr/0024-smaltml-scope-and-architecture.md)
  (the model this record follows), smaltml [ADR 0034](https://github.com/stenhoor/smaltml/blob/main/docs/adr/0034-where-auto-runs.md)
  (where `'auto'` runs), smaltml [ADR 0035](imported/smaltml/0035-simd-kernels-in-a-plugin.md) (replaceable kernels;
  SIMD in a plugin), smaltdb [ADR 0036](imported/smaltdb/0036-user-defined-functions.md) (functions that plugins
  register), [ADR 0037](0037-releases-independent-packages.md) (independent releases);
  [DECISIONS.md](../DECISIONS.md): "WebAssembly: plugins only", "Outside code: ask first, and plugins carry it"

## Context

The owner (2026-11-17): smaltlp is a linear programming module for the smalt family, in TypeScript. It takes PuLP,
scipy, GLPK, CLP, HiGHS and OR-Tools as its spiritual ancestors, fits the family as smaltml does, and can use WebGPU
and SIMD to go faster.

### What exists (researched 2026-11-17)

Sources: the npm and PyPI registries and the projects' licence files, read on 2026-11-17. Points marked *(assumption)*
were not confirmed in the session; netlib.org, miplib.zib.de, docs.scipy.org and arxiv.org couldn't be reached from it.

- **The references.**
  - **HiGHS** (MIT): dual and primal simplex, interior point, PDLP and branch-and-cut. It reads LP and MPS files.
  - **scipy**'s `linprog(c, A_ub, b_ub, A_eq, b_eq, bounds, method='highs', …, integrality)` and
    `milp(c, *, integrality, bounds, constraints, options)` both run HiGHS. Read from scipy's source on 2026-11-17:
    `milp` takes `LinearConstraint(A, lb, ub)` rows; both return `x`, `fun`, `status`, `message` and `success`; status
    0 is optimal, 1 a limit, 2 infeasible and 3 unbounded.
  - **PuLP** (MIT; 4.2.0, 2026-10-22) builds models from expressions: `LpProblem`, `LpVariable`, `lpSum`, `prob += …`.
  - **OR-Tools** (Apache-2.0) has GLOP (simplex), PDLP and CP-SAT. **COIN-OR CLP and CBC** are EPL-2.0. **GLPK** is
    GPL-3.0.
- **In JavaScript today.**
  - **Pure JavaScript solvers** are dense and meant for small models: `javascript-lp-solver` (1.0.3, Unlicense) and
    `yalps` (0.6.4, MIT). YALPS's own README says it is for "hundreds of variables or constraints". Neither has
    presolve, an LU-based revised simplex, MPS files or strong integer methods (from their READMEs).
  - **Serious solvers** come only as other projects compiled to WebAssembly: `highs` (HiGHS, 1.15.4, MIT, 4.0 MB)
    and `glpk.js` (5.0.0, GPL-3.0).
  - Nothing found is a sparse, typed TypeScript solver with presolve, MPS files and solid integer methods.
- **LP on the GPU.**
  - PDLP, a first-order method (restarted PDHG), is how LP runs on GPUs: cuPDLP, cuPDLPx, NVIDIA cuOpt and HiGHS's GPU
    PDLP.
  - It converges to about 1e-4 relative, against simplex's 1e-7 to 1e-9 *(assumption)*.
  - **Precision:** cuOpt documents a single-precision mode (from a search snippet). No study of fully 32-bit PDLP's
    accuracy was found.
  - **Crossover:** COPT runs a crossover to a simplex basis afterwards by default, and cuOpt as an option (snippets).
  - **WebGPU:** sundial-core (a Rust crate) runs PDHG in 32-bit WGSL and checks the result in 64-bit on the CPU
    (snippet). Its own documentation says a CPU simplex beats it on small LPs.
- **What the family has.**
  - smaltml is the pattern: a TypeScript library in the Kits layer, the CPU path in 64-bit floats as the product,
    WebGPU as a measured accelerator, and compare tests against Python saved as fixtures.
  - WebAssembly is allowed only in plugins, in the circumstances the owner confirmed. smaltml-simd is the one such
    plugin. Its kernels belong to smaltml's own kernel contract (`useKernels`, `KERNELS_VERSION` 1). (As of
    2026-11-18, WebGPU and SIMD are in smaltkit: section 8.)
  - smaltdb ADR 0036 lets plugins register functions that every query language can call.

## Decision

The owner's answers of 2026-11-17, unless a point says otherwise.

### 1. Scope: LP, then MIP

| Phase | What it holds |
| --- | --- |
| 1 | Continuous linear programs: a bounded dual simplex, presolve and scaling, duals and reduced costs, MPS and LP files, the modelling layer and the matrix functions (sections 3 to 5) |
| 2 | Mixed-integer programs: branch-and-bound warm-started from the dual simplex, reliability branching, presolve for integers and basic primal heuristics |
| 3 | Cutting planes (such as Gomory, mixed-integer rounding and cover cuts) and more heuristics |
| Later, measured | WebGPU (section 7) and replaceable kernels with a SIMD plugin (section 8) |

- **Sizes: small to medium,** up to about 10⁴ to 10⁵ variables and constraints, in line with "small data in the
  browser". Larger models aren't refused; they aren't what the design is tuned for.
- **Left out:** quadratic programs and constraint programming (as in CP-SAT). Either needs a new ADR.

### 2. References: behaviour, not code

As smaltml ADR 0024 section 2 has it:

- **scipy and HiGHS** are the reference for results and status codes. **PuLP** is the reference for the modelling
  layer.
- **Our own code,** from the documentation and the published papers, such as Koberstein's dual simplex, Forrest and
  Goldfarb's steepest edge, Suhl and Suhl's LU, Huangfu and Hall's updates, and Achterberg for integer methods.
- No code is copied or translated from HiGHS, PuLP, OR-Tools, CLP, CBC or GLPK, whatever their licence. GLPK's GPL
  would rule it out anyway.

### 3. API: two front doors, one core

- **A modelling layer in PuLP's style,** typed: variables, linear expressions, constraints and objectives, by name.
- **scipy's matrix functions:** `linprog()` and `milp()`, over matrices and bounds, with scipy's parameters in
  camelCase and its status codes.
- **One core:** both build the same internal standard form, and one solver runs it.
- **Asynchronous calls over a synchronous core,** as smaltml section 5 has it:
  - every solve has a `*Sync` twin;
  - the core touches neither the DOM nor Node;
  - a worker entry point runs solves off a page's main thread.

### 4. The CPU path is the product

- **64-bit floats,** in every browser the family supports and in Node.
- **First algorithm: a bounded revised dual simplex** with sparse LU factorization and updates, presolve and scaling.
  It gives vertex solutions, duals and reduced costs, and the warm starts that MIP (phase 2) and crossover
  (section 7) need.
- Interior point isn't in the first phases ("Still to decide", item 2).

### 5. Placement: a library, standing alone

- **A library in the Kits layer.** It is not a smaltdb plugin, and the engine knows nothing of optimization.
- **It stands alone:** no dependency on smaltml. Its needs are sparse where smaltml's are dense. The two share
  patterns (async over sync, the worker, `device`, kernel contracts), not code, and this is revisited if real
  duplication appears.
- **Reaching the apps and languages:**
  - **TypeScript first:** smaltnb notebooks, smaltui's TypeScript tabs and flows call it directly.
  - **Models from tables:** helpers build a model from a smaltdb query result or a smaltdf frame (costs, bounds,
    coefficient rows), and write solutions back as columns. smaltdb and smaltdf are optional peers, used only by
    those helpers.
  - **Files:** MPS (free and fixed) and CPLEX-LP, read and written, so models move to and from other solvers.
  - **Queries:** any database route is a function a plugin registers through smaltdb ADR 0036, as predict is. It is a
    scalar function returning a struct (smaltdb ADR 0020): for example `solve_lp(model)` giving
    `{ status, objective, x }`, with `x` a list a query can explode into rows. How it takes its model is settled in
    that plugin's own record.

### 6. Verification: compared with Python once, then kept as tests

As smaltml ADR 0024 section 7 has it:

- **Compare tests:** each case runs through smaltlp and through **highspy and scipy** in Python. **PuLP** also checks
  the modelling layer, through its LP and MPS files.
- **Agreement:** objective values and solutions within each case's stated tolerance. Where a problem has several
  optimal solutions, the check is on the objective and on feasibility, not on `x`.
- **Fixtures:** the results are saved as fixtures and checked in CI without Python.
- **Outside code:** highspy, scipy and PuLP are pinned in `compare/requirements.txt`, for development only. The owner
  chose them on 2026-11-17. The first compare PR states their exact versions and licences.
- **Test instances:**
  - Our own small models, written by hand or generated, are committed with their fixtures.
  - Netlib LPs and MIPLIB 2017 "easy" instances are downloaded in CI, by URL and checksum, and are not committed until
    their terms are checked.

### 7. WebGPU: a later phase, kept only where measured

- **The method:** PDLP in 32-bit floats on the GPU, a check of the result in 64-bit on the CPU, then a crossover or
  polish by the dual simplex.
- **A research spike comes first.**
- **Measured, like smaltml ADR 0034:** `device: 'auto'` uses the GPU only where it is measured at least 2× faster at
  the stated tolerance. A GPU result that fails the 64-bit check is never returned as optimal.
- **On smaltkit's compute** (amended 2026-11-18, section 8): the GPU kernels, in WGSL, are smaltlp's; smaltkit's
  WebGPU module (`smaltkit/gpu`, smaltkit#52, open on 2026-11-18) gives the device and runs them.

### 8. Kernels: smaltlp writes them, smaltkit's compute runs them

Amended 2026-11-18 (the owner): "WebGPU and SIMD have been migrated to smaltkit; smaltlp will retain responsibility
for writing kernels to execute by the kit's compute efforts." This replaces the first version's own kernel contract
with a plugin decided later (smaltml-simd grown, a sibling plugin, or none).

- **smaltkit's compute runs the kernels.** `smaltkit/simd` (on smaltkit's `main`: `Code`, `ModuleBuilder`, `Room`,
  `simdSupport`, f64x2 math) writes and runs WebAssembly SIMD from TypeScript; `smaltkit/gpu` will do the same for
  WebGPU.
- **smaltlp writes its own kernels,** for its own steps, with those modules: the SIMD instructions and the WGSL. They
  live in smaltlp, behind an optional entry point (such as `@smaltworks/smaltlp/kernels`) that the core solver never
  imports (the owner, 2026-11-18).
- **The core stays TypeScript and works alone.** Without the kernels' entry, or where the engine has no SIMD or no
  WebGPU, every solve runs in plain TypeScript, as now. The kernels give results within rounding of the TypeScript
  path, checked by tests, as smaltml's are.
- **smaltkit is an optional peer dependency** (the owner, 2026-11-18): the kernels' entry imports it; the core doesn't,
  so an app without smaltkit still solves. smaltlp keeps no runtime dependencies.
- **Where SIMD helps** *(assumption: to be measured)*. Sparse simplex work is mostly memory-bound and gains little.
  Dense steps gain more, such as an interior point's A·D·Aᵀ, a weighted Gram matrix. No kernel ships without a
  benchmark that shows it gains.

### 9. Package, repository and brand

- **Package:** one package, `@smaltworks/smaltlp`, starting at 0.14.0, with an entry point per area:
  - `/model`, `/lp`, `/mip`, `/io` (MPS and LP files), `/tables`, `/worker`;
  - `/gpu` when section 7 is built.
- **No runtime dependencies.**
- **Repository:** `stenhoor/smaltlp`, following STANDARDS.md:
  - tests with `node --test`, Node `>=22.18`;
  - typecheck on TypeScript 7, `dist/` emitted by TypeScript 6.0.3 for one more release;
  - MIT licence, "smaltworks contributors".
- **Brand: its own, like smaltml.** smaltlp is a product people choose and use directly. smalt-brand picks its words,
  mark and colour; its Kits layer is assumed until then.

## Consequences

- **A gap filled:** the browser gets a sparse LP and MIP solver in TypeScript with no WebAssembly and no outside code.
  Today that means HiGHS compiled to WebAssembly (4.0 MB) or GPL code.
- **The hard part is numerical.** A dual simplex that is robust on Netlib needs sparse LU with stable updates,
  presolve, scaling and careful tolerances. Phase 1 is split into a first slice and the rest ("Still to decide",
  item 3).
- **Results agree with HiGHS within stated tolerances,** not bit for bit, and not always at the same vertex.
- **The query route waits on smaltdb's plugin functions** (ADR 0036) and on a plugin of its own. The TypeScript
  library comes first.
- **Brand work in smalt-brand:** the words, the mark, the site kit copy and a page on smaltworks.dev.

## Considered

- **Wrap HiGHS compiled to WebAssembly** (`highs`). This is the fastest route to a strong solver. It is ruled out by
  "WebAssembly: plugins only", criterion 3: our own code, no WebAssembly builds of other libraries.
- **Extend `yalps` or `javascript-lp-solver`:** both are dense tableau designs, and are outside code.
- **Interior point or PDLP first:** neither gives a basis, which MIP warm starts and sensitivity need. The dual simplex
  is HiGHS's default for LP *(assumption)* and the base the later phases build on.
- **Depend on smaltml or a shared core package:** this couples two kits' releases and contracts for little shared
  code.
- **A table function for queries:** reads better (`FROM solve_lp(…)`), but ADR 0036's table functions aren't designed
  yet. A struct exploded into rows gets the same result now.

## Decided 2026-11-19

These were "Still to decide" in the first version. The owner answered items 1, 2 and 4 on 2026-11-19; items 3 and 5
record what has happened since.

1. **A performance goal: tracked, no target yet.** smaltlp is timed beside HiGHS on every solver change, and the
   ratios are published (smaltlp's `docs/research/benchmarks.md`), with no pass or fail number. A target is set once
   there is a public benchmark set to hold it to: Netlib and MIPLIB, downloaded in CI (section 6), which this
   environment's network doesn't reach yet. On 2026-11-19 smaltlp took 1.4 to 2.4 times HiGHS's time with its
   defaults on smaltlp's own LP benchmark, and 2.7 to 10 times HiGHS's serial dual simplex without presolve.
2. **Interior point: later, as the first test of a SIMD kernel.** After integer programming is solid, a research spike
   on an interior-point method, the first place a `smaltkit/simd` kernel is measured (its A·D·Aᵀ, section 8). It
   ships only where its benchmark shows a gain.
3. **The first slice** shipped as proposed, then the rest of phase 1 and the start of phase 2, one pull request at a
   time (stenhoor/smaltlp#1 to #14): the modelling layer in PuLP 4.2's style, `linprog()`, the bounded dual simplex,
   a sparse LU, presolve, MPS, the compare tests and the docs site; then `milp()` with branch-and-bound, warm starts,
   and cheaper nodes. Gomory cuts at the root were built and measured slower on every benchmark problem; the owner
   chose to make each node cheaper first (2026-11-19), and cuts come back only when a benchmark shows they gain.
4. **The status model: scipy's single code, as built.** `linprog()` and `milp()` give scipy's codes; the modelling
   layer gives PuLP 4.2's. No separate termination and solution statuses underneath, unless a caller needs more (a
   feasible solution from a limit, say).
5. **The brand** is smalt-brand's, decided 2026-11-17 (smalt-brand DECISIONS.md, "smaltlp"): "Small. Alternative.
   Optimal.", the Aim mark, the Kits layer's jade.
