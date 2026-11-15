# Exceptions

Where a repository departs from [STANDARDS.md](STANDARDS.md) or another rule here, on purpose. Each exception names the
rule, the repository, why, who approved it, and when it ends, if it does. A departure that isn't listed here is drift,
to be fixed.

| Repository | Rule | Exception | Why | Approved | Ends |
| --- | --- | --- | --- | --- | --- |
| smaltfl | One ADR sequence ([adr/README.md](adr/README.md#numbering)) | Numbers its ADRs in its own sequence, from 0001, and its maintainers decide its design | A shared project with its own maintainers ([smalt-brand BRAND.md, smaltfl](https://github.com/stenhoor/smalt-brand/blob/main/BRAND.md#smaltfl)) | Recorded in smalt-brand before 2026-11-15 | – |
| smaltty | TypeScript: the latest stable, within one minor release ([STANDARDS.md](STANDARDS.md#typescript-and-browsers)) | Keeps TypeScript 6.0.3 for its compiler API (its differential tests) | TypeScript 7 has no stable JavaScript API yet | The owner, 2026-11-12 ([smaltty ADR 0030](adr/imported/smaltty/0030-changing-smaltty-without-breaking-its-consumers.md)) | When TypeScript 7 has a stable JavaScript API (7.1 at the earliest) |
| smaltdb-sucrase | All standards | Retired; kept for reference, not maintained | Replaced by smaltty ([DECISIONS.md](DECISIONS.md#smaltty-typescript-without-outside-code)) | The owner, 2026-11-08 | – |
| smaltcy, smaltdb, smaltdb-external, smaltdb-io, smaltdb-node, smaltdb-parquet, smaltdf, smaltfl, smaltgl, smaltml, smaltql, smaltts | Tests run with `node --test` ([STANDARDS.md](STANDARDS.md#node-tests-branches-licence-and-dependencies)) | Unit tests run with vitest | Already written with vitest; moving them all at once isn't worth it | The owner, 2026-11-15 | Per repository, when its tests are being changed anyway |

Proposed exceptions waiting for the owner are in the latest [review](reviews/).
