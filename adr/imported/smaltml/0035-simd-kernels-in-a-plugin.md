# ADR 0035: WebAssembly SIMD kernels, in a plugin: smaltml-simd

> **Copied** from [smaltml/docs/adr/0035-simd-kernels-in-a-plugin.md](https://github.com/stenhoor/smaltml/blob/1852dd470ec50700a4d86c2094a710ed54f5a654/docs/adr/0035-simd-kernels-in-a-plugin.md) (commit `1852dd4`) on 2026-11-17, replacing the copy of that morning (commit `bdbe801`) after stenhoor/smaltml#45 amended it. The source stays in place until its removal PR. Relative links were rewritten to resolve from smaltworks.

- **Status:** accepted, 2026-11-17 (the owner). The owner's direction and decisions of 2026-11-15 are recorded under
  Decision (items 2 and 10); those of 2026-11-17 in items 3, 5 and 8 and under "Still to decide", items 1 and 4. Items 3 and
  8 were amended on acceptance, as each says.
- **Date:** 2026-11-15
- **Amends:** [ADR 0024](0024-smaltml-scope-and-architecture.md): section 8 ("Where it is hardest") and "Still to
  decide", item 5 (whether smaltml may write its own WebAssembly kernels).

## Context

ADR 0024 makes smaltml native TypeScript: no runtime dependencies, no WebAssembly builds of other libraries, no code
loaded from another site. It left one question open for the case where TypeScript can't reach Python's speed: whether
smaltml may write its own WebAssembly kernels.

Where smaltml stands (the [benchmarks](https://github.com/stenhoor/smaltml/blob/main/docs/research/benchmarks.md), 200,000 × 50, one machine):

- **Behind scikit-learn on the fits built on XᵀX.** On one thread, Ridge takes 2.7 times as long as scikit-learn,
  Lasso 2.3 times and LogisticRegression with `newton-cholesky` 2.0 times. scikit-learn computes XᵀX with BLAS, which
  uses the processor's SIMD instructions; smaltml's kernel (`gramPart`, blocked, a 2 × 4 tile) is scalar TypeScript.
- **JavaScript has no SIMD.** The engines compile a loop over a `Float64Array` to scalar machine code, and the
  SIMD.js proposal was withdrawn. WebAssembly has had 128-bit SIMD in every major browser since 2021–2023. The one
  route to SIMD from a browser's CPU is WebAssembly.
- **No generic route exists.** No plugin or library runs arbitrary JavaScript on SIMD: JavaScript's dynamic types,
  objects and closures don't compile to vector instructions. Projects that use SIMD from the browser run code
  written for it: their own kernels (DuckDB-Wasm, the WebAssembly backends of TensorFlow.js and ONNX Runtime) or a
  restricted language (AssemblyScript). A smalt plugin can do the same: kernels for a few fixed operations, not
  arbitrary code.

### The probe (2026-11-15)

To see whether it is worth it, `gramPart`'s work was written as a WebAssembly SIMD kernel, without a toolchain: about
120 lines of TypeScript that write the WebAssembly bytes directly. It does exactly what `gramPart` does: the same
blocks of 512 rows, copied into a buffer (here, the WebAssembly memory), and the same 2 × 4 tile of the lower triangle,
but over two rows at a time (`f64x2`). Columns are padded to a multiple of 4 and odd blocks to an even number of rows,
with zeros.

Node 22.23 (V8 12.4), a 4-core Xeon VM; the median of 11 runs, in three rounds:

| X | `gramPart` (TypeScript) | WebAssembly SIMD | SIMD with fused multiply-add |
| --- | ---: | ---: | ---: |
| 200,000 × 50 (the benchmark data) | 165–180 ms | 36–50 ms, 3.5–4.6× | 33–38 ms, 4.7–5.1× |
| 50,000 × 200 (random) | 517–530 ms | 132–144 ms, 3.7–3.9× | 115–126 ms, 4.1–4.6× |
| 10,000 × 20 (the benchmark data) | 1.6–3.0 ms | 0.4–0.8 ms, about 3.6× | about 3.6× |

- **Agreement:** within 2 × 10⁻¹⁴ of each entry on the benchmark data. On the random 50,000 × 200 matrix the largest
  relative difference is 1.2 × 10⁻¹¹, on off-diagonal entries close to 0 (where the sums cancel).
- **Why more than 2×,** when `f64x2` carries only two numbers: compiled WebAssembly also drops the bounds and type
  checks the JavaScript engine makes on every array read.
- **Fused multiply-add** is the relaxed SIMD instruction `f64x2.relaxed_madd` (Chrome 114 and later; not in every
  browser). Whether it fuses depends on the processor, so its results can differ in the last bits from one machine to
  another. How far is measured below ("How far results move").
- **Noise:** the VM varies by about ±30% between rounds; every round showed the same ratios.
- **For scale:** XᵀX for 200,000 × 50 in about 35 ms, against scikit-learn's whole Ridge fit (XᵀX, Xᵀy and the solve)
  in 75 ms on one thread.

### How far results move

Any change to the order of the sums moves the last bits, and the solve can magnify that by up to the condition number of
XᵀX. Measured on the probe (the XᵀX of centred X, then Ridge's Cholesky solve on it), as the largest difference
relative to each entry's scale (√(A_jj·A_kk)) and to the largest coefficient:

| X | cond(XᵀX) | XᵀX: plugin vs smaltml | XᵀX: fused vs plain | Coefficients: plugin vs smaltml | Coefficients: fused vs plain |
| --- | ---: | ---: | ---: | ---: | ---: |
| 200,000 × 50, the benchmark data | 9 × 10² | 1.5 × 10⁻¹⁵ | 1.9 × 10⁻¹⁶ | 5.8 × 10⁻¹⁶ | 3.8 × 10⁻¹⁶ |
| 50,000 × 200, random | 1.3 | 1.1 × 10⁻¹⁵ | 2.2 × 10⁻¹⁶ | 6.4 × 10⁻¹⁶ | 2.6 × 10⁻¹⁶ |
| 100,000 × 10, two columns 10⁻⁶ apart, alpha 1 | 4 × 10¹² | 1.5 × 10⁻¹⁵ | 2.2 × 10⁻¹⁶ | 2.5 × 10⁻¹² | 9.8 × 10⁻¹⁴ |
| the same, alpha 0 (no penalty) | 4 × 10¹² | 1.5 × 10⁻¹⁵ | 2.2 × 10⁻¹⁶ | 8.7 × 10⁻⁴ | 5.6 × 10⁻¹² |

- **Fused multiply-add moves results less than the plugin itself does.** Between two machines (one that fuses, one
  that doesn't), XᵀX differs by about one rounding step (2 × 10⁻¹⁶); turning the plugin on at all moves it about
  7 times as much, from summing in another order.
- **On well-conditioned data, coefficients move by under 10⁻¹⁵.** The compare tests' tolerances against scikit-learn
  are 10⁻⁹ to 10⁻⁵, a million times wider and more.
- **On nearly collinear data without a penalty, any change of order is magnified.** Coefficients moved by 9 × 10⁻⁴
  between the plugin and smaltml's own kernel, and 6 × 10⁻¹² between fused and plain. Such a fit has no well-defined
  answer to more digits than that whichever kernel runs: scikit-learn's own result on it also depends on its BLAS.
  smaltml's LinearRegression refines its solution and falls back to QR on such data, which the probe didn't.

The probe isn't in the repository. Its code and benchmark move to smaltml-simd as that repository's first pull request
(below), so the numbers above link to a benchmark anyone can run, as the brand requires.

## Decision

1. **The core stays TypeScript.** `@smaltworks/smaltml` contains no WebAssembly, writes none, and loads none. Every
   model, kernel and result in it is TypeScript, as ADR 0024 says. This is the owner's principle (2026-11-15):
   TypeScript is the product's language; WebAssembly is not.

2. **WebAssembly SIMD goes in a plugin, `smaltml-simd`** (owner, 2026-11-15): its own repository,
   `stenhoor/smaltml-simd`, and package, `@smaltworks/smaltml-simd`, as the family's plugins have (smaltdb
   [ADR 0012](https://smaltdb.dev/adr/0012-plugins); the brand's rule that outside code a feature needs goes in a
   named, optional plugin). An app that doesn't install it runs exactly as today. It is shown as smaltdb's plugins are
   (item 10).

3. **smaltml gets one extension point, in TypeScript: replaceable kernels.** The plugin can only swap the CPU kernels
   smaltml already has, with the same signatures:

   - `useKernels({ name, version, kernels: { gramPart, ... }, threads })` in `@smaltworks/smaltml/core` sets them;
     `useKernels(null)` puts smaltml's own back. The solvers and models don't change.
   - **The kernel contract has a version** (`KERNELS_VERSION`, 1), so the plugin can grow with smaltml's later phases
     (the owner, 2026-11-17: the plugin serves smaltml's current and future needs, without painting itself into a
     box). Adding a kernel to the contract keeps the version; changing what one takes or gives raises it, and a set
     written for another version isn't used. Amended on acceptance: the first draft had kernels only, unversioned.
   - A kernel the plugin doesn't give stays smaltml's. A plugin kernel that fails a check at start (a small product
     against the TypeScript kernel) isn't used.
   - The plugin is used only where the app turns it on: `await useSimd()` from `@smaltworks/smaltml-simd`. Nothing in
     smaltml looks for it.

4. **The kernels: the ones the probe shows gain,** the arithmetic-bound ones:
   - `gramPart` (XᵀX), for LinearRegression, Ridge, Lasso and ElasticNet;
   - the weighted Gram matrix, `gramPart` with row weights, for `newton-cholesky`;
   - the CPU matrix product behind `matMul`.

   Not the memory-bound ones: Xw, Xᵀv and the scalers read each value once and spend their time waiting for memory,
   which SIMD doesn't change. Not the logistic objective until measured: it spends much of its time in `exp`, which
   WebAssembly doesn't have as a SIMD instruction. Measured on 2026-11-17, and added: its row steps, with the
   plugin's own exp (amended on acceptance, "Still to decide", item 4).

5. **The plugin writes its WebAssembly from TypeScript, in its own code.** Its kernels are TypeScript functions that
   emit the WebAssembly bytes at run time, as the probe does: no compiler toolchain, no `.wasm` file built elsewhere,
   no outside package, nothing loaded from another site. The code that writes each instruction is in the repository,
   to read and review. Considered on 2026-11-17: Rust compiled to WebAssembly, for its optimizing compiler and SIMD
   intrinsics. The owner kept TypeScript: the kernels are small, their speed comes from the blocking chosen by hand,
   and a second language and toolchain would cost more than they give. smaltml's side doesn't depend on the plugin's
   language, so the choice can be revisited inside smaltml-simd, by an amendment here, if the kernels outgrow an
   emitter.

6. **Plain SIMD by default; fused multiply-add only when asked** (`useSimd({ fusedMultiplyAdd: true })`, where the
   browser has relaxed SIMD). Plain SIMD gives the same result on every machine; fused multiply-add can differ in the
   last bits between processors.

7. **Results with the plugin are close to smaltml's own, not equal.** SIMD sums in a different order, so results move
   in the last bits (about 10⁻¹⁴ relative on the probe). What holds with the plugin on:
   - every compare case against scikit-learn passes within its tolerance; the plugin's CI runs smaltml's compare
     tests with the plugin on;
   - an asynchronous call gives exactly its `*Sync` twin's result, and a fit on any number of threads gives exactly
     `fitSync`'s, as long as every path uses the plugin's kernels (item 8). Plain SIMD keeps this on every machine;
     fused multiply-add keeps it on one machine.

8. **Threads use the plugin too.** The thread pool's workers run the same kernels, or a threaded fit would differ
   from `fitSync`. The kernel set names a module (`threads: { module, options }`); smaltml's own workers import it
   when the pool starts and use the set its default export gives, as the main thread does. Each worker compiles the
   kernels once. A worker that can't load the set, or ends up with other kernels, keeps fits on one thread; so does a
   set without `threads`. Amended on acceptance (2026-11-17): the first draft had the plugin ship its own workers'
   script, smaltml's with its kernels, set by `configureThreads({ script })`. That copy of smaltml's worker would
   drift from smaltml's protocol with every smaltml release; a module the workers import keeps the plugin free of
   smaltml's internals.

9. **Where WebAssembly SIMD isn't available,** `useSimd()` reports it (it resolves to `false`) and smaltml's own
   kernels stay in use. Node has it; browsers have had it since Chrome 91, Firefox 89 and Safari 16.4.

10. **Shown as smaltdb's plugins are** (owner, 2026-11-15; smalt-brand BRAND.md, "Plugins"). smaltml-simd is an
    infrastructure plugin: smaltml's brand and a plain name, "smaltml SIMD", with no mark or colour of its own. Its
    README carries the Official plugin badge with smaltml's mark. Its docs are a plugin site (BRAND.md,
    "Documentation": guide per plugin, API reference), and smaltml's site lists it on a plugins page, as smaltdb.dev
    lists smaltdb's.

## Consequences

- **The fits built on XᵀX should get 2 to 4 times faster** where the app turns the plugin on, by the probe; the
  benchmarks will say by how much for each model, against scikit-learn, as for every claim.
- **The core keeps its promise:** one language, no WebAssembly, no dependencies. The cost is a second repository with
  its own CI, release and docs, and an extension point in the core to keep stable.
- **Two kinds of result:** with and without the plugin, close but not bit for bit. Saved models are the same either
  way; only the numbers in their last bits differ.
- **Users choose speed knowingly:** the docs say what the plugin changes (speed, last bits) and what it doesn't (the
  models, the API, agreement with scikit-learn).
- **The same approach could serve other repositories later** (column arithmetic in smaltdf or smaltdb), from
  smaltml-simd or a shared plugin in smaltkit. Not decided here.

## Steps

1. This record, then the owner's decisions on "Still to decide".
2. Done (2026-11-17): smaltml: `useKernels` in the core, with tests that a replaced kernel is used by every model, on one thread and on
   several, and that `useKernels(null)` restores smaltml's own. TypeScript only.
3. The repository `stenhoor/smaltml-simd` (the owner creates it), in the family's layout: its brand from
   smalt-brand, CI, docs.
4. smaltml-simd's first pull request: the probe's kernel as `gramPart`, the benchmark, and smaltml's compare tests
   run with it on. Then the weighted Gram matrix and the matrix product, each with its benchmark.
5. smaltml's docs: a page on the plugin in the hardware guide, linking to smaltml-simd's site.

## Still to decide

1. ~~**Fused multiply-add**~~: decided 2026-11-17 (the owner): offered as an option, off by default (item 6).
   Measured ("How far results move"): fused and plain differ by about one rounding step in XᵀX and under 10⁻¹⁵ in
   well-conditioned coefficients, less than the plugin itself moves them, for 6–30% more speed on the probe.
2. ~~**The brand**~~: decided 2026-11-15, shown as smaltdb's plugins are (item 10).
3. **Threads and SIMD together:** the probe ran on one thread. Whether the gains multiply on 4 threads, or memory
   bandwidth caps them, is to be measured in step 4.
4. ~~**The logistic objective**~~: decided 2026-11-17 (the owner): build it, after measuring.
   - **Measured** (`node bench/exp-share.mjs`: smaltml's objectives as they are, and with `Math.exp`, `Math.log1p` and
     `Math.log` replaced by cheap arithmetic; the same 4-core VM, several runs):

     | Data | Objective | ms per evaluation | Share of exp, log1p, log | A fit's speed-up if they cost nothing |
     | --- | --- | ---: | ---: | ---: |
     | 10k × 20 | binary | 0.65–1.1 | 47–49% | about 1.9× |
     | 10k × 20 | 3 classes | 1.6–2.6 | 22–36% | about 1.3–1.5× |
     | 200k × 50 | binary | 28–34 | 18–23% | about 1.27× |
     | 200k × 50 | 3 classes | 72–73 | 12–15% | about 1.15× |

     An `lbfgs` fit is almost all objective evaluations (100 iterations of 29 ms in a 3.4 s fit, 200k × 50, 2
     classes). The share falls as columns grow: a row costs one to three of these functions, and multiply-adds that
     grow with the columns. One run gave −17% for 200k × 50, 3 classes, against 12–15% in every other: the VM's noise.
   - **The owner's decision:** build it anyway, for the gain on narrow data.
   - **How:** two more kernels in the contract (still version 1: adding kernels doesn't change it),
     `binaryLogisticRows` and `multinomialLogisticRows`: each objective's step between Xw and Xᵀr, which computes the
     loss and replaces the scores by their residuals. smaltml's own are its loops as they were, so its own results
     don't change, bit for bit; the running loss goes in, so the sum keeps its order. smaltml-simd brings its own
     f64x2 exp, log1p and log (a polynomial and a series each, within a few rounding steps of `Math`'s), so logistic
     fits with the plugin are close to smaltml's own, not equal, as the others are.
