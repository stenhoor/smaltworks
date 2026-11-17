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
| 2026-11-09 | ADR 0025 and 0026: smaltai (proposed) | [0025](adr/imported/smalt-brand/0025-smaltai-agent-gateways-and-actions.md), [0026](adr/imported/smalt-brand/0026-smaltai-in-browser-llm.md) |
| 2026-11-17 | ADR 0039: smaltlp (proposed); its "Still to decide" items, and the ADR 0037 number used twice ([adr/README.md](adr/README.md#numbering)) | [0039](adr/0039-smaltlp-scope-and-architecture.md) |
