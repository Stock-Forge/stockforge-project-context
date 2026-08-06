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
| Day | **Day 3 (complete) — next: Day 4** |
| Status | Day 3 done and pushed; Day 4 ready to start |
| Current repository | stockforge-contracts (work repo, pushed) + stockforge-project-context (state repo) |
| Current branch | main |
| Current commit | stockforge-project-context: `c3141d4` → new closeout commit; stockforge-contracts: `e1d65cb` |

---

## WHERE WE ARE (day by day)

| Day | Topic | Status |
|---|---|---|
| 0 | Project foundation (master prompt + context files) | ✅ done |
| 1 | Phase 0 architecture proposal | ✅ done |
| 1.5 | Architecture approved + day-by-day guide | ✅ done |
| 2 | GitHub org `Stock-Forge` + repo created + pushed + git round-trip practiced | ✅ done |
| 3 | `stockforge-contracts` — API & event contracts (OpenAPI v1.0.0 + 7 events) | ✅ done |
| 4 | `stockforge-web` — React trading UI scaffold | ⏭️ **NEXT** |
| 5 | `stockforge-api` — Spring Boot gateway scaffold | planned |
| 6-7 | `stockforge-auth` — register/login (bcrypt + JWT), roles | planned |
| 8-9 | `stockforge-order-service` — order lifecycle (in-memory), risk mock | planned |
| 10+ | PostgreSQL → risk → matching → market-data → portfolio → notification → integration → … → HFT (roadmap §21) | planned |

---

## What Day 3 completed

- GitHub repo **`stockforge-contracts`** created (empty) by the user; local repo created as its own git repo (sibling to project-context, never nested).
- Built contract-first deliverables:
  - `contracts/openapi.yaml` — OpenAPI 3.0.3, 10 paths (auth, market-data, orders, portfolio), 14 schemas, JWT bearer security. Validated as parseable YAML.
  - `contracts/events/` — 7 event contracts (OrderCreated, OrderAccepted, OrderRejected, OrderExecuted, OrderCancelled, PositionUpdated, MarketPriceUpdated), each with schema, producer→consumer, downstream effects, failure handling (idempotency, ordering, DLQ notes), plus `INDEX.md` (topics, partition keys, cross-cutting rules).
  - `README.md` — why contract-first, layout, contract rules, status.
- Committed `e1d65cb` and pushed to `Stock-Forge/stockforge-contracts` (tracking set).

## What is deliberately NOT done

- **Device B clone still unverified** (user-owned) — both repos should be cloned there:
  ```
  git clone https://github.com/Stock-Forge/stockforge-project-context.git
  git clone https://github.com/Stock-Forge/stockforge-contracts.git
  ```
- No application code (starts Day 4 with `stockforge-web`).
- Contract validation tooling (swagger-cli / schema registry) and contract tests not yet added (planned when services arrive / Phase 14).
- `stockforge-contracts` has no CI yet (Phase 14).

## Incomplete work (open items carried forward)

```
- Device B clone verification (user-owned, do anytime).
- Contract tooling + contract tests (deferred to service phases / Phase 14).
```

---

## NEXT DAY PROMPT — DAY 4: `stockforge-web`

**How to start:** copy-paste the FULL content of
`stockforge-project-context\project-context\START_OF_DAY.md` into the new AI session.
Below is the day-specific plan the AI must follow.

### Day 4 — React trading UI scaffold

**Goal:** a working (ugly) web app: Vite + React + TypeScript, page shell with a login
placeholder and a dashboard that shows "connected" state. First real frontend repo.

**Why / production thinking:** the browser is where users interact; a frontend repo owns
its UI, its own CI, its own deploy even though it talks to many backends. We scaffold
frontend first so there is always something visible; the API comes Day 5.

**Step 1 — You (manual, on GitHub):**
- In org `Stock-Forge`, create empty repo `stockforge-web` (no README/.gitignore/license).
- Make sure Node.js LTS is installed (`node --version` — Node v24 is installed on this machine).

**Step 2 — AI session does:**
1. Startup protocol (pull here, read context/state/prompts, reconcile).
2. Create local folder `C:\CODE\HFT Application\stockforge-web` — its OWN git repo.
3. Scaffold with Vite: `npm create vite@latest . -- --template react-ts`
   - explain `package.json`, `tsconfig`, `index.html`, `src/` structure
   - page shell: simple login placeholder + dashboard showing "API not connected yet"
   - `npm run dev` works at `localhost:5173`
4. Commit + push to `Stock-Forge/stockforge-web` (message describes the change).
5. **CENTRAL-STATE RULE:** update state HERE — `CURRENT_STATE.md` (Day 4 done → pointer to
   Day 5), `SESSION_PROMPTS.md` (Session 4 entry), `DAY_BY_DAY_GUIDE.md` (mark Day 4 done),
   `CHANGELOG.md`, `PROJECT_CONTEXT.md` if changed.
6. Commit + push this repo. Verify BOTH pushes.

**Expected result:** `stockforge-web` on GitHub with a runnable React+TS app; this repo's
state points at Day 5.

**Environment note:** Node.js v24.14.1 already installed on this machine.

---

## GIT STATUS (verify on the other device)

```
git status        # clean in both repos
git log --oneline # project-context → closeout commit; contracts → e1d65cb
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
