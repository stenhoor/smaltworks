# Decision queue

What waits on the owner, oldest first. Updated when a review runs, never by monitoring. An item leaves when it is
decided; the decision goes to [DECISIONS.md](DECISIONS.md) or an ADR.

## Held open on purpose

| Since | Item | Where things stand |
| --- | --- | --- |
| 2026-11-15 | **Running code from other sites (future plugins).** How apps and the console may load plugin code that isn't ours: smaltbi ADR 0008 (proposed) loads report plugins from other sites, pinned by hash, after the reader agrees | Held in tension until there is better direction. **Safety is the primary concern;** the answer also has to be flexible. Until decided, no app loads code from another site at run time, and smaltbi 0008 stays proposed |

## Waiting for a decision

| Since | Item | Record |
| --- | --- | --- |
| 2026-11-08 | smaltfl ADRs 0010–0013 and 0015 (proposed) | smaltfl's own; its maintainers decide its design |
| 2026-11-09 | ADR 0025 and 0026: smaltai (proposed) | [0025](adr/imported/smalt-brand/0025-smaltai-agent-gateways-and-actions.md), [0026](adr/imported/smalt-brand/0026-smaltai-in-browser-llm.md) |
| 2026-11-15 | **The circumstances in which a plugin may use WebAssembly**: the criteria drafted in DECISIONS.md, "WebAssembly: plugins only" | [DECISIONS.md](DECISIONS.md#webassembly-plugins-only) |
| 2026-11-15 | ADR 0035, smaltml-simd: its status, once the WebAssembly criteria are confirmed | [0035](adr/imported/smaltml/0035-simd-kernels-in-a-plugin.md) |
| 2026-11-15 | smaltgl ADRs 0001 and 0003 still say "a product of its own" and `stenhoor/smalt-geo`; amend to the kits layer and `stenhoor/smaltgl` | [adr/README.md, Conflicts](adr/README.md#conflicts) |
