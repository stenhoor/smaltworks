# Orchestrator / Worker Handoff — Architecture Overview (Draft)

Status: **Draft for discussion** · Protocol version: `v1` (proposed)

This document captures the handoff model agreed so far between an
orchestrator agent and worker agents running as Claude cloud sessions, and
lists the open issues that remain before the protocol can be finalised.

---

## 1. Principles

1. **Communication is by GitHub issues.** Orchestrators open task issues in
   the target repository; all coordination happens on that issue.
2. **Authority lives in authored, append-only comments.** Project board
   fields are derived views, never a source of truth.
3. **Identity decisions are made in code**, from GitHub's recorded author
   (`user.login`), never by the model reading text.
4. **Ownership is two-level.** Coarse: issue author + assignee = owning
   collaborator. Fine: the claim comment = owning agent.
5. **Workers act only on work from their own collaborator's identity.**
   Collaborators are trusted; the filter prevents cross-collaborator mistakes.
6. **Protocol logic lives in a CLI (`smalt`)**; skills are thin wrappers that
   say *when* to call it; hooks block bypassing it.

---

## 2. Components

```mermaid
flowchart LR
    subgraph C["Collaborator identity (e.g. stenhoor)"]
        H["Human collaborator"]
        subgraph O["Orchestrator session<br/>AGENT_ID=stenhoor/orchestrator-1"]
            OS["Skills: plan-adr, create-task, sweep"]
            OC["smalt CLI"]
        end
        subgraph W["Worker sessions (one per environment)<br/>AGENT_ID=stenhoor/worker-api-1"]
            WS["Skills: poll, claim, heartbeat, release"]
            WC["smalt CLI"]
            HK["Hooks: block raw issue reads"]
        end
        ST["Scheduled tasks<br/>(wake workers)"]
    end

    subgraph SW["smaltworks repo"]
        ADR["ADRs"]
        PL["Plugin marketplace:<br/>smaltworks-protocol<br/>(skills + smalt CLI + hooks)"]
        SPEC["Protocol spec"]
        RW["Reusable Actions workflow:<br/>board sync"]
    end

    subgraph TR["Target repositories"]
        IS["Task issues<br/>(locked, assigned to owner)"]
        CM["Comment log<br/>smaltworks:claim / release /<br/>reassign / heartbeat"]
        GA["Actions: board sync<br/>(uses RW + PAT)"]
        PR["Branches / PRs"]
    end

    PB[("Shared project board<br/>(user-owned)<br/>Status, AGENT_WORKER,<br/>HEARTBEAT, ADR, Milestone")]

    H -->|directs| O
    OS --> OC
    O -->|reads| ADR
    OC -->|create + lock + assign| IS
    OC -->|reassign / sweep| CM
    ST -->|wake| W
    WS --> WC
    WC -->|filtered read| IS
    WC -->|append| CM
    W -->|work| PR
    PL -.->|installed via .claude/settings.json| O
    PL -.->|installed via .claude/settings.json| W
    CM -->|issue_comment event| GA
    RW -.-> GA
    GA -->|derive fields| PB
    O -->|reads status| PB
```

---

## 3. Task lifecycle — create, claim, work

```mermaid
sequenceDiagram
    autonumber
    participant Orc as Orchestrator<br/>(stenhoor/orchestrator-1)
    participant Iss as Task issue<br/>(target repo)
    participant Act as Actions board-sync
    participant Brd as Project board
    participant Sch as Scheduled task
    participant Wk as Worker<br/>(stenhoor/worker-api-1/cse_…)

    Orc->>Iss: Open issue from template (ORCHESTRATOR header)
    Orc->>Iss: Lock, assign to stenhoor
    Orc->>Iss: Comment: smaltworks:fingerprint v1 (hash of body)
    Orc->>Brd: Add item (Status=Ready)

    Sch->>Wk: Wake: run /smaltworks:poll
    Wk->>Iss: smalt poll (filter: author+assignee = owner, no active claim)
    Wk->>Iss: smalt verify-fingerprint
    Wk->>Iss: Comment: smaltworks:claim v1 AGENT_WORKER=…
    Note over Wk: wait N seconds
    Wk->>Iss: smalt resolve-claim (earliest valid claim since last release/reassign)
    alt won
        Wk->>Iss: Comment: smaltworks:status v1 in-progress
        Iss-->>Act: issue_comment event
        Act->>Brd: Status=In progress, AGENT_WORKER=stenhoor/worker-api-1
        loop while working
            Wk->>Iss: smaltworks:heartbeat v1 (at checkpoints)
            Act->>Brd: HEARTBEAT=…
            Wk->>Iss: smalt resolve-claim (re-check before push/PR)
        end
        Wk->>Iss: Completion flow (TBD)
    else lost
        Note over Wk: Stop. Post nothing further.
    end

    Orc->>Brd: sweep: stale HEARTBEAT?
    Orc->>Iss: Comment: smaltworks:reassign v1 TO=open
```

---

## 4. Task states

```mermaid
stateDiagram-v2
    [*] --> Ready: orchestrator creates issue
    Ready --> Claimed: claim comment posted
    Claimed --> Ready: claim lost / release
    Claimed --> InProgress: claim verified
    InProgress --> Ready: release / reassign(open)
    InProgress --> Claimed: reassign(TO=agent)
    InProgress --> NeedsInfo: question raised (TBD)
    NeedsInfo --> InProgress: answered by owner (TBD)
    InProgress --> InReview: PR opened (TBD)
    InReview --> Done: accepted (TBD)
    InReview --> InProgress: changes requested (TBD)
    Done --> [*]
```

---

## 5. Decisions so far

| # | Area | Decision |
|---|---|---|
| D1 | Channel | GitHub issues in the target repo, opened only by orchestrators |
| D2 | Protection | Task issues locked on creation (write-access only) |
| D3 | Trust | Collaborators trusted; workers act only on content authored by their owner's login, filtered in code |
| D4 | Agent identity | `AGENT_ID` env var (`<login>/<role>`) + `CLAUDE_CODE_REMOTE_SESSION_ID`; refuse to claim if either missing |
| D5 | Claim | Append-only `smaltworks:claim v1` comment; earliest valid claim since last release/reassign wins; wait-then-verify |
| D6 | Release / reassign | `smaltworks:release v1` (holder only), `smaltworks:reassign v1` (orchestrator only) open a new claim round |
| D7 | Board | One shared user-owned Projects v2 board; fields are derived, comments are authoritative |
| D8 | Fields | `Status` (single select), `AGENT_WORKER` (single select, role), `HEARTBEAT` (text, `id @ timestamp`) |
| D9 | Implementation | Protocol logic in a `smalt` CLI; thin skills; hooks block bypass; shipped as a plugin from smaltworks |

---

## 6. Open issues

### Blockers — must verify before building

| # | Issue | Why it matters | Proposed next step |
|---|---|---|---|
| B1 | **GitHub API access from cloud sessions.** The provided GitHub tools cover issues/PRs but not Projects v2 (GraphQL); direct API access is limited. | Determines whether `smalt` can run inside sessions or only in Actions, and whether agents can write board fields. | Spike: from a cloud session, list issue comments with author metadata and update a test board item. |
| B2 | **Plugin distribution to cloud sessions.** Can a session auto-install a plugin from the smaltworks repo (declared in a target repo's `.claude/settings.json`)? | Without it, skills/CLI/hooks must be copied into every repo and will drift. | Spike: declare the marketplace in a test repo, start a session, confirm skills and hooks load. |
| B3 | **Board automation credentials.** Actions can't use `GITHUB_TOKEN` on a user-owned board; needs a PAT (likely classic, `project` scope). | A broad, long-lived token per repo; secret management and rotation. | Confirm whether fine-grained PATs work on user boards; otherwise document scope and rotation. |
| B4 | **Shared API rate limit.** Every agent of a collaborator uses the same identity and so the same GitHub rate limit. | Polling many repos with many workers can exhaust it and stall everything. | Size polling cadence; prefer orchestrator → worker messaging over polling; cache in `smalt`. |

### Design decisions still open

| # | Issue | Options / notes |
|---|---|---|
| O1 | **Worker triggering.** | Scheduled polling (hourly minimum is typical), orchestrator messages the worker session directly, or both (message as fast path, poll as safety net). |
| O2 | **Session lifecycle.** Who starts worker sessions, and how is "one live session per `AGENT_ID`" ensured? | One environment per role with `AGENT_ID` set; orchestrator starts/restarts sessions; detect duplicates via session id in claims. |
| O3 | **Role takeover.** Can a new session with the same `AGENT_ID` resume its role's in-progress task, or must the orchestrator reassign? | Recommend: orchestrator reassigns (explicit, auditable). |
| O4 | **Zombie holders.** After a reassign, the old session may still be alive and keep working. | `smalt resolve-claim` re-check required before every push, PR, or status change (fencing). Define which actions are checkpoints. |
| O5 | **Timing parameters.** Claim wait N, heartbeat cadence, stale timeout. | Use GitHub server timestamps (comment `created_at`), not session clocks, for staleness. |
| O6 | **Partial work on reassign.** What happens to the old holder's branch/PR? | Convention: branch name per task; new holder continues or restarts; release comment records branch state. |
| O7 | **Fingerprint and legitimate edits.** Orchestrator may need to amend a task. | Orchestrator posts a new `smaltworks:fingerprint` (or `amend`) comment; workers re-verify and re-read. |
| O8 | **Valid orchestrators.** How does a worker know an `ORCHESTRATOR` header is legitimate, and how is an orchestrator handed off? | Registry of orchestrator IDs per collaborator in smaltworks config; orchestrator handoff via the same release/claim pattern on a control issue? |
| O9 | **Task issue template.** | Proposed fields: ADR ref, milestone, repo, acceptance criteria, dependencies, branch convention, expected output, protocol version. |
| O10 | **Cross-repo dependencies.** | Orchestrator holds blocked tasks back vs. opens them with `Status=Blocked` and dependency links. |
| O11 | **ADR home and mapping.** | ADRs move to `smaltworks/adr/<repo>/NNNN-*.md`? Is ADR → milestone → task stored in files, board fields, or both? GitHub milestones are per-repo. |
| O12 | **Comment-format status changes.** Should every status change be a comment (authoritative) and the board purely derived? | Recommended, for consistency with D7; costs comment volume. |
| O13 | **Hook coverage.** Exact list of tools/commands to block (GitHub MCP issue/comment reads, `gh`, `curl` to the API). | Needs to be tested; incomplete blocking silently reopens the injection path. |
| O14 | **Protocol versioning and skew.** Collaborators may run different plugin versions. | Version in every comment (`v1`); `smalt` rejects unknown versions; upgrade policy. |

### Flows not yet designed

| # | Flow |
|---|---|
| F1 | Completion: returning a finished work item (PR link, evidence, who accepts, closes issue) |
| F2 | Needs-info: worker raises a question; who answers; how the answer is authenticated; resume |
| F3 | Failure / abandon: worker cannot complete; error reporting |
| F4 | Cancellation: orchestrator withdraws a task mid-flight |
| F5 | Orchestrator handoff: moving an ADR's work between orchestrators |

### Accepted / residual risks

| # | Risk | Mitigation |
|---|---|---|
| R1 | Prompt injection via task inputs (code, docs, dependencies, PR reviews). | Treat as data; filter PR review comments by author too; narrow repo access per worker. |
| R2 | A steered agent acting with its collaborator's full GitHub access. | Least-privilege repo attachment per session; only orchestrators create tasks. |
| R3 | Board field changes are not author-attributed. | Fields are display only; orchestrator reconciles from comments. |
| R4 | User-owned board is a single point of ownership. | Board is rebuildable from comment logs; migrate when the org exists. |
