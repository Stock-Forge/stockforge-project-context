# CURRENT_STATE.md

> Current progress state. Updated after every meaningful session.
> Read this first (after PROJECT_CONTEXT.md) when starting a new AI session.
> This file always shows: where we are (day by day) + the prompt to begin the next day.

---

## Snapshot

| Field | Value |
|---|---|
| Date | 2026-08-06 |
| Phase | Phase 1 — Git/GitHub organization and repository foundation |
| Day | **Day 3 — starting (contracts)** |
| Status | Previous day complete and pushed; next day ready to start |
| Current repository | stockforge-project-context (state repo) — Day 3 work repo: `stockforge-contracts` |
| Current branch | main |
| Current commit | `cb9af31` (pushed to `origin/main`) |

---

## WHERE WE ARE (day by day)

| Day | Topic | Status |
|---|---|---|
| 0 | Project foundation (master prompt + context files) | ✅ done |
| 1 | Phase 0 architecture proposal | ✅ done |
| 1.5 | Architecture approved + day-by-day guide | ✅ done |
| 2 | GitHub org `Stock-Forge` + repo created + pushed + git round-trip practiced | ✅ done |
| 3 | `stockforge-contracts` — API & event contracts | ⏭️ **NEXT** |
| 4 | `stockforge-web` — React trading UI scaffold | planned |
| 5 | `stockforge-api` — Spring Boot gateway scaffold | planned |
| 6-7 | `stockforge-auth` — register/login (bcrypt + JWT), roles | planned |
| 8-9 | `stockforge-order-service` — order lifecycle (in-memory), risk mock | planned |
| 10+ | PostgreSQL → risk → matching → market-data → portfolio → notification → integration → … → HFT (roadmap §21) | planned |

---

## What the previous session (Day 2) completed

- GitHub org **`Stock-Forge`** created (hyphenated; product name stays `StockForge`).
- Repo `stockforge-project-context` created on GitHub, remote `origin` added, `main` pushed + tracking set.
- Docs updated to actual org name; 30-minute roadmap expanded to Day 39+ (`PROJECT_CONTEXT.md` §21).
- Git round-trip practiced: branch → commit → push -u → merge → push → delete branch.
- `CHANGELOG.md` created. Commit `cb9af31` on `origin/main` (working tree clean).

## What is deliberately NOT done

- **Device B clone + round-trip not yet exercised** — user action on the other machine:
  ```
  cd C:\CODE
  git clone https://github.com/Stock-Forge/stockforge-project-context.git
  cd stockforge-project-context
  git log --oneline      # expect 9 commits ending in cb9af31
  ```
- No application code (starts Day 3 with contracts).
- Branch protection + PR workflow not enabled (planned Phase 14 / GitHub Actions).

## Incomplete work (open items carried into Day 3)

```
- Device B clone verification (user-owned, do anytime).
- stockforge-contracts repo does not exist yet (user creates on GitHub at Day 3 start).
```

---

## NEXT DAY PROMPT — DAY 3: `stockforge-contracts`

**How to start:** copy-paste the FULL content of
`stockforge-project-context\project-context\START_OF_DAY.md` into the new AI session.
Below is the day-specific plan the AI must follow.

### Day 3 — API & event contracts (Phase 1 foundation)

**Goal:** create the contracts repo — the *agreement* between services: what endpoints
exist, what requests/responses look like, what Kafka events carry. No application code.

**Why / production thinking:** 10 services can silently break each other unless the
interface is pinned and versioned. Enterprises are **contract-first**: the API spec is
written before the service, so frontend and backend are built in parallel against the
same contract (OpenAPI, AsyncAPI/event registry, contract tests in CI later).

**Step 1 — You (manual, on GitHub):**
- In org `Stock-Forge`, create an empty repo `stockforge-contracts` (no README,
  no .gitignore, no license) — same flow as A.3 in the guide.

**Step 2 — AI session does:**
1. Startup protocol (pull here, read context/state/prompts, reconcile).
2. Create local folder `C:\CODE\HFT Application\stockforge-contracts` — its OWN git
   repo (sibling to this one, never nested).
3. Build:
   - `contracts/openapi.yaml` — auth + orders + portfolio endpoints (from
     PROJECT_CONTEXT.md §7 catalogue) with basic request/response schemas
   - `contracts/events/` — the 7 Kafka events (OrderCreated, OrderAccepted,
     OrderRejected, OrderExecuted, OrderCancelled, PositionUpdated,
     MarketPriceUpdated) with producer/consumer/payload notes
   - `README.md` explaining contract-first, layout, and versioning rules
   - Explain every important section while writing.
4. Commit + push to `Stock-Forge/stockforge-contracts` (message describes the change).
5. **CENTRAL-STATE RULE:** update state HERE — `CURRENT_STATE.md` (mark Day 3 done,
   move pointer to Day 4), `SESSION_PROMPTS.md` (add Session 3 entry),
   `DAY_BY_DAY_GUIDE.md` (mark Day 3 done), `CHANGELOG.md` (add Day 3 line),
   `PROJECT_CONTEXT.md` if architecture changed.
6. Commit + push this repo. Verify BOTH pushes (`git status -sb` up to date).

**Expected result:** `stockforge-contracts` on GitHub with `openapi.yaml`, `events/`
(7 event files), and a README; this repo's state points at Day 4.

**Expected commit for this repo after closeout:** a new commit on `main` (current `cb9af31`).

---

## GIT STATUS (verify on the other device)

```
git status        # clean
git branch -a     # only main (local + origin/main)
git log --oneline # 9 commits, HEAD = cb9af31
```

---

## HOW TO BEGIN ANY FUTURE DAY (same procedure every day)

**Rule:** copy-paste the FULL content of
`stockforge-project-context\project-context\START_OF_DAY.md` into the new AI session.
That file IS the begin-day prompt — it does not need editing each day. It makes the AI:
reconstruct (pull + read context/state/latest prompt + reconcile) → work ONE ~30-minute
day per `DAY_BY_DAY_GUIDE.md` → run the mandatory end-of-day sequence → push BOTH repos.

**CENTRAL-STATE RULE (baked into START_OF_DAY.md):** project state is updated ONLY in
this repository (`stockforge-project-context`) — `CURRENT_STATE.md`,
`SESSION_PROMPTS.md`, `PROJECT_CONTEXT.md`, `DAY_BY_DAY_GUIDE.md`, `CHANGELOG.md`.
We often work in a DIFFERENT repo that day (e.g. `stockforge-contracts`); that repo
gets only its own code, tests, and README. State updates happen HERE, always, and are
pushed HERE. Both repos are committed and pushed every day.

**Quick-start summary for the new AI (START_OF_DAY.md expands this):**

```
StockForge. New session, zero memory.

1. git pull in stockforge-project-context (and in any other repo being touched).
2. Read PROJECT_CONTEXT.md, CURRENT_STATE.md, latest entry in SESSION_PROMPTS.md.
3. Check git status / branch / log -5. Reconcile state vs prompt vs repo. Investigate
   before coding if they disagree. Report: Project reconstructed + phase/day/repo/
   branch/last task/incomplete/next task.
4. Work ONE ~30-minute day per DAY_BY_DAY_GUIDE.md. Teach while building. If not
   finished, record incomplete work openly (reason/state/remains/next).
5. End of day (mandatory):
   - run tests
   - update the working repo's own README
   - CENTRAL-STATE RULE: update PROJECT_CONTEXT.md (if architecture changed),
     CURRENT_STATE.md, SESSION_PROMPTS.md, DAY_BY_DAY_GUIDE.md, CHANGELOG.md
     HERE in stockforge-project-context — never in the working repo
   - git status + git diff review
   - git add + git commit (message describes the change, not the day)
   - git push in BOTH repos; verify (git status -sb up to date)
   - never claim saved until pushes are verified
```
