# ADR 0040: The SIMD kernels in smaltml, as their own entry point

> **Copied** from [smaltml/docs/adr/0040-simd-kernels-in-smaltml.md](https://github.com/stenhoor/smaltml/blob/be6ac14b518ace19bb28aaf34108126f76229ce6/docs/adr/0040-simd-kernels-in-smaltml.md) (commit `be6ac14`) on 2026-11-18. The source stays in place until its removal PR. Relative links were rewritten to resolve from smaltworks.

- **Status:** accepted, 2026-11-18 (the owner). The owner asked to move the kernels out of smaltml-simd ("I want to
  remove all dependencies to that repo") and chose, of three places, smaltml's own entry point.
- **Date:** 2026-11-18
- **Number:** 0040, the next free number in the family repositories this session can read (0039 is smaltml's trees and
  forests). Renumber if another repository took 0040.
- **Supersedes:** [ADR 0035](https://github.com/stenhoor/smaltml/blob/be6ac14b518ace19bb28aaf34108126f76229ce6/docs/adr/0035-simd-kernels-in-a-plugin.md)'s items 1, 3 (where the kernels live and how smaltml finds
  them), 9 and 10, and its plugin repository. Its other items carry over, as below.

## Context

ADR 0035 put smaltml's WebAssembly SIMD kernels in a plugin, smaltml-simd, so the core stayed TypeScript with no
WebAssembly, and smaltml looked for the plugin where it was installed (amended 2026-11-17). On 2026-11-18 the
WebAssembly writer moved to smaltkit, as `smaltkit/simd` (ADR 0035, item 5, amended), so the plugin held only smaltml's
kernels, written against smaltml's own kernel contract and released in step with it. The owner then asked to remove
smaltml's dependencies on the plugin's repository. Three places were offered for the kernels:

1. **smaltml, as its own entry point**, `@smaltworks/smaltml/simd` (chosen);
2. smaltml's core, always on, with no entry of its own;
3. smaltkit, beside its writer.

## Decision

1. **The kernels are smaltml's, in `src/simd/`, published as `@smaltworks/smaltml/simd`:** `gramPart`, `matMul`, the
   logistic row steps and `logisticProducts`, with `useSimd()` and `simdSupport()`. smaltml-simd's code moves here as
   it was; the WebAssembly the kernels write is the same, byte for byte.
2. **The core still has no WebAssembly.** Only the `simd` entry writes WebAssembly, and only the `simd` area may import
   `smaltkit/simd` (test/modules.test.ts). The core reaches it as it reached the plugin: `kernelsReady()` imports
   `@smaltworks/smaltml/simd` by name, through a variable, so a bundler doesn't follow it, and Node finds it by the
   package's self-reference. Where it loads and the engine has WebAssembly SIMD, smaltml turns the kernels on at the
   first asynchronous fit or `matMul`, as it turned the plugin on; where it doesn't (a bundled app that leaves the entry
   out), smaltml's own kernels stay, and the app imports the entry and calls `useSimd()` itself.
3. **No dependency for users.** The build folds `smaltkit/simd` into `dist/` (esbuild bundles each entry), so
   smaltkit stays a development dependency, pinned to a commit, and smaltml keeps no runtime dependencies. The public
   types don't name smaltkit: `SimdSupport` is smaltml's own.
4. **The thread pool's workers** load the kernel set from `dist/smaltml-simd-kernels.js`, built as one file with no
   imports, beside smaltml's other worker scripts (`@smaltworks/smaltml/simd-kernels-script`), as they loaded the
   plugin's (ADR 0035, item 8). A bundled app copies it beside its bundle and names it (`useSimd({ kernelsModule })`).
5. **Carried over from ADR 0035:** the kernel contract and its version (item 3's `useKernels`, `KERNELS_VERSION`), the
   kernels written from TypeScript at run time with our own writer (item 5, as amended), plain SIMD by default and
   fused multiply-add only when asked (item 6), results close to smaltml's own and not equal, every compare case within
   its tolerance, threaded fits equal to `fitSync` (item 7), and the logistic row steps off by default (its "Still to
   decide", item 4).
6. **Tests:** smaltml's test suite runs on its own TypeScript kernels (`test/setup.ts`), so its results stay
   bit-exact. `npm run test:simd` runs the compare, parallel and hardware tests with every SIMD kernel on, plain and
   with fused multiply-add, as the plugin's CI did; CI runs it. The kernels' own tests (against plain loops, on odd
   sizes) moved from the plugin to `test/simd-*.test.ts`.
7. **smaltml-simd is retired.** Its last pull request, stenhoor/smaltml-simd#8, isn't needed: its work is here. What
   becomes of the repository (archived, or kept with a notice) is the owner's to decide.

## WebGPU too (the owner, 2026-11-18)

The same split for WebGPU: what any smalt product could use moves to smaltkit, as `smaltkit/gpu`
(stenhoor/smaltkit#52): the device, asked for once and again after it is lost; compute pipelines from WGSL source,
kept per device; and the tiled matrix product in 32-bit floats, its shader unchanged. smaltml keeps what is its own:
`device: 'auto' | 'cpu' | 'gpu'` and its threshold (ADR 0034), its error codes, and the logistic objective's shader.
Like the SIMD kernels, the shaders are source (WGSL text) that the browser compiles where it runs, not a precompiled
binary, and the build folds smaltkit's module into `dist/`, so there is still no runtime dependency.

## Consequences

- One repository and one release for smaltml and its kernels: no pins between them, no plugin site, no second CI.
- **The family's WebAssembly rule holds** (the owner, 2026-11-18): the kernels are TypeScript that writes WebAssembly
  bytes at run time, compiled live by the engine, not a precompiled binary, so they fit the family's practice
  ("WebAssembly: plugins only", smaltworks DECISIONS.md, 2026-11-15: our own code from source, optional, narrow,
  behind a TypeScript interface). That record's wording, which names plugins, is to be brought in line there.
- Apps get SIMD without installing anything; an app that wants smaltml's own kernels calls `useKernels(null)`.
- The plugin extension point stays: `useKernels` and the contract are unchanged, for other plugins.
