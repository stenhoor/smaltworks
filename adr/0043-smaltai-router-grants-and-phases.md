# ADR 0043: smaltai: a router to the user's own gateways, fine-grained grants, and three phases

- **Status:** accepted (2026-11-22, the owner, in an interview the same day)
- **Date:** 2026-11-22
- **Tier:** 1 (project-wide)
- **Repositories:** smaltai (new), smaltbi (first app), smaltkit (the chat panel), later smaltnb, smaltui, smaltfl
- **Builds on:** [ADR 0025](imported/smalt-brand/0025-smaltai-agent-gateways-and-actions.md) (gateways, actions,
  permissions, context documents) and [ADR 0026](imported/smalt-brand/0026-smaltai-in-browser-llm.md) (the in-browser
  engine), both proposed 2026-11-09.
- **Supersedes in ADR 0025:** section 2's single gateway (now several endpoints, section 2 here), section 5's four grants
  (now section 4 here), section 9 and "Still to decide" 1 (the first app is smaltbi) and "Still to decide" 2 (grants are
  remembered). The rest of ADR 0025 stands, including section 3, "No keys, ever". ADR 0026 stays proposed, as phase 2.

## Context

smaltai lets the smalt apps use an AI model: a router to a local or a cloud model, a performant in-browser model later,
and an agent in each app that builds queries, reports, notebooks and flows through the app's own actions.

The question the owner set first was where keys can live safely. Research on 2026-11-22 found:

- **A page can call some providers directly,** but only by holding the key in JavaScript. Anthropic's API allows browser
  calls with the `anthropic-dangerous-direct-browser-access` header, and Gemini's API allows the page's origin (both
  probed with requests); OpenAI's SDK disables browser use by default and warns that users can extract the key (its
  README). No provider found offers short-lived browser tokens without a backend to mint them.
- **A browser can't hide a key from code on its own origin.** A key sent in a header exists as plain text in
  JavaScript; a non-extractable WebCrypto key can't be exported but can still be used by same-origin script (the
  WebCrypto spec).
- **Every smalt app shares one origin, `stenhoor.github.io`,** with the docs sites, and code users write runs there
  (smaltui's TypeScript tabs, smaltnb's kernel, the docs playgrounds). `github.io` is on the Public Suffix List (read);
  that project sites share one origin follows from how origins work (inference).
- **Local gateways can be reached from the page,** with conditions: Chrome (142 and later) asks the user once before a
  public page reaches a local address (search result, not read first-hand); Ollama needs `OLLAMA_ORIGINS` set to the
  apps' origin (Ollama's FAQ); llama-server allows any origin by default (its README); LM Studio's CORS is a setting
  (search result). **Safari may block `http://localhost` from an https page** (unconfirmed; see "Still to check").

The owner's aim: keep the project out of as many security concerns as possible, with no bad press and no being pulled
into supporting users' security problems.

## Decision

### 1. Keys live only in a local gateway the user runs

ADR 0025's rule stands: **smalt never sees or holds a key.** A cloud model is reached only through a gateway on the
user's machine or network that holds the key. No browser vault, no key in any smalt page or storage.

### 2. Gateways are existing tools, and the user's responsibility

smaltai ships **no gateway of its own** in this effort. It documents how to use existing ones, with the settings smalt
needs (allowing the apps' origin, the endpoint's address): Ollama, llama-server and LM Studio for local models, and a
key-holding proxy such as LiteLLM for cloud models. Managing keys, access and spending stays with the user and their
tool.

The router in the browser knows **several endpoints**. The user registers each one (its address and format, never a
key) and picks the endpoint and model for each chat; smaltai remembers the choice. There is no hidden routing between
them.

### 3. The in-browser model is a separate path

The in-browser engine (ADR 0026) doesn't go through the router. It is an alternative path to a local model that offers
the same interface as a gateway endpoint (a chat request with tools, streamed back), so the agent doesn't care which it
uses.

### 4. Grants: fine-grained, remembered, revocable

What the agent may see and do is granted per endpoint and per database:

- **Data levels**, one per database: **schema only** (names and types, the open document's outline); **schema and
  statistics** (counts, distinct counts, minimum and maximum, null rates; no rows); **schema and sample rows** (a sample
  of a size the user sets, with query results capped to it); **full data**.
- **Narrowed per table and column:** the user can exclude tables or columns from whatever level is granted. Excluded
  columns never leave the browser.
- **Actions**, for smaltbi, the first app: **change the report** (measures, visuals, pages, filters, through `DocStore`,
  so each change can be undone); **run queries** (what comes back to the model follows the data level); **create views
  and materialized views**. **Changing data is not recommended for smaltbi**, and is off unless the user enables it.
  **Configuration tables always need the user's explicit approval**, whatever is granted.
- **Grants decide what runs:** within its grants, the agent acts without asking each time; outside them, it asks.
- **Remembered** per endpoint and database in the browser, shown together in a grants panel where any can be revoked.

### 5. Everything is logged, locally

Every request and response (full text, so the user sees exactly what left the browser), every action with its input and
result, every grant change and every approval is logged in the browser, viewable in the app and exportable as a file.
Nothing is sent anywhere.

### 6. Defences on by default

- **No remote content in the chat:** images and links in a model's output are never loaded automatically; links show as
  text to open deliberately. (A common way to carry data out is a URL the page fetches.)
- **Data is marked as data:** rows and file contents sent to the model are wrapped and labelled as untrusted data, never
  as instructions.
- **Tokens shown:** tokens used per request and per session. Budgets themselves stay the gateway's job.
- **A kill switch** stops the agent mid-task and cancels pending actions.

### 7. Agent inside the app; smaltbi first

The agent is embedded in each app (ADR 0025's design: the app's typed actions offered as tools, every action shown,
undo where the app has it). Actions are designed so they could be offered to outside agents over MCP later; that is not
part of this effort. **smaltbi is first**, then the others (smaltnb, smaltui, smaltfl).

### 8. No AI on Safari for now

Where the browser can't reach a local gateway, the AI button says why and names the browsers that work. Safari gets AI
later through the in-browser engine where WebGPU allows. This follows confirming Safari's behaviour ("Still to check").

### 9. Three phases

1. **The router and the app agent:** endpoints, grants, logging, defences, smaltbi's actions and the chat panel, with
   documented existing gateways.
2. **The in-browser engine** (ADR 0026).
3. **A gateway of our own,** last, if it is still wanted.

## Consequences

- smalt holds no keys and runs no service, so it has no credentials to lose and no gateway to secure.
- Users who want a cloud model install and configure a gateway themselves, and meet Chrome's one-time prompt.
- Safari users get no AI until phase 2.
- smaltbi needs its actions declared, and an agent guide: smaltbi has no docs for agents yet, while smaltql, smaltdf,
  smaltcy, smaltkit and smaltml do (checked 2026-11-15).

## Still to check

1. **Safari and `http://localhost` from an https page:** confirm first-hand (WebKit bug 279249 reports Safari 18 beta
   blocking it; search result).
2. **Chrome's local network prompt** (version and wording) and **LM Studio's CORS default:** confirm from their own
   documentation.
3. **OpenAI's CORS behaviour** for direct calls: not needed under section 1, recorded for completeness.

## Considered

- **A key in the app's own browser storage:** rejected; any code on the shared origin could read it.
- **A key vault on a separate origin** (an iframe reached by `postMessage`): no install, but it puts a key in a browser
  and makes smalt responsible for it.
- **A browser extension holding keys:** Chrome-centric (Firefox doesn't support page-to-extension messaging the same
  way, per MDN; search result) and a store listing to maintain.
- **Our own gateway now:** moved to the last phase, to keep the project out of security support.
- **Rules-based routing** in the browser (e.g. rows only to local models): left to the user's choice of endpoint.
