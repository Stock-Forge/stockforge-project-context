# CURRENT_STATE.md

> Current progress state. Updated after every meaningful session.
> Read this first (after PROJECT_CONTEXT.md) when starting a new AI session.
> This file always shows: where we are (day by day) + the prompt to begin the next day.

---

## Snapshot

| Field | Value |
|---|---|
| Date | 2026-08-07 |
| Phase | Phase 1 — Git/GitHub organization and repository foundation |
| Day | **Day 5 (started, BLOCKED at tooling gate — deferred to personal PC)** |
| Status | Day 4 done & pushed. Day 5 NOT built: this device has Java 17 + no admin; JDK 21 setup moves to the personal PC (full admin). State + issues-log + prompt features updated & pushed here. |
| Current repository | stockforge-project-context (state repo) — Day 5 build happens on the personal PC |
| Current branch | main |
| Current commit | stockforge-project-context: new closeout commit; stockforge-web: `8e7d075` |

---

## WHERE WE ARE (day by day)

| Day | Topic | Status |
|---|---|---|
| 0 | Project foundation (master prompt + context files) | ✅ done |
| 1 | Phase 0 architecture proposal | ✅ done |
| 1.5 | Architecture approved + day-by-day guide | ✅ done |
| 2 | GitHub org `Stock-Forge` + repo created + pushed + git round-trip practiced | ✅ done |
| 3 | `stockforge-contracts` — API & event contracts (OpenAPI v1.0.0 + 7 events) | ✅ done |
| 4 | `stockforge-web` — React trading UI scaffold | ✅ done |
| 5 | `stockforge-api` — Spring Boot gateway scaffold | ⏭️ **NEXT (on personal PC)** |
| 6-7 | `stockforge-auth` — register/login (bcrypt + JWT), roles | planned |
| 8-9 | `stockforge-order-service` — order lifecycle (in-memory), risk mock | planned |
| 10+ | PostgreSQL → risk → matching → market-data → portfolio → notification → integration → … → HFT (roadmap §21) | planned |

---

## Day 5 status — started, stopped at tooling gate

**What happened today (2026-08-07):**
- Started the Day 5 startup protocol (pull, reconstruct, tooling check).
- This device reported **Java 17.0.12** (stack wants **JDK 21**) and **no Maven**.
  Maven is not an issue — the Spring Boot scaffold ships the **Maven Wrapper** (`mvnw`)
  which pins its own version per repo.
- This device has **no complete admin rights**, so the JDK 21 installer is not viable here.
- Decision (user): **do the portable-Temurin-JDK-21 setup and the Day 5 build on the
  personal PC** (full admin). Portable ZIP extraction + user-level `JAVA_HOME`/`PATH`
  also works without admin if needed later.
- Prompt improvements shipped this session (already pushed `19166d2`): full-detail day
  briefing (steps + tech stack + Zerodha/Groww comparison) and a new `ISSUES_LOG.md`
  with auto-update at end of every session.

**Open items carried forward:**
```
- Install JDK 21 on the personal PC (Temurin 21; installer there, or portable ZIP).
- Clone stockforge-project-context + stockforge-web on the personal PC, then run Day 5
  (stockforge-api scaffold) there per the Day 5 prompt below.
- This device (no admin): can still build frontend (Node OK) and edit state files;
  backend JVM work happens on the personal PC until containers (Phase 12-13) make
  toolchains device-independent.
```

## What Day 4 completed

- GitHub repo **`stockforge-web`** created (empty) by the user; local repo created as its own git repo (sibling, never nested).
- Scaffolded **Vite + React + TypeScript** app (`npm create vite@latest . -- --template react-ts`); explained `package.json`, `tsconfig*`, `index.html`, `src/` structure.
- Built the page shell: header with **StockForge** brand, **login placeholder** (disabled form), **dashboard** with two status cards showing "API not connected yet".
- Replaced template demo styles with a clean dark StockForge theme; removed unused template assets.
- Verified: `npm run lint` (oxlint) ✅, `npm run build` (tsc -b + vite build) ✅, `npm run dev` served HTTP 200 at `http://localhost:5173` with `#root` mounted.
- Committed `8e7d075` and pushed to `Stock-Forge/stockforge-web` (tracking set).

## What is deliberately NOT done

- **Device B clone still unverified** (user-owned) — clone this repo on the other device:
  ```
  git clone https://github.com/Stock-Forge/stockforge-web.git
  ```
- No API wiring — the UI makes zero HTTP calls yet (stockforge-api is Day 5).
- No routing (login/dashboard are on one page), no real auth, no state management, no WebSocket.
- No frontend tests or CI yet (Phase 14).

## Incomplete work (open items carried forward)

```
- Device B clone verification (user-owned, do anytime).
- Contract tooling + contract tests (deferred to service phases / Phase 14).
- stockforge-web is a static shell — wiring to stockforge-api starts Day 5.
```

---

## NEXT DAY PROMPT — DAY 5: `stockforge-api`

**How to start:** copy-paste the FULL content of
`stockforge-project-context\project-context\START_OF_DAY.md` into the new AI session.
Below is the day-specific plan the AI must follow.

### Day 5 — Spring Boot gateway scaffold

**Goal:** a Spring Boot app with `/api/health`, structured logging, a correlation ID
filter, and a placeholder that talks to nothing yet. First backend repo.

**Why / production thinking:** the API is the single client-facing door; everything in
production goes through it — auth, validation, rate limiting, tracing. We learn Spring
Boot's request lifecycle here; the concepts (filters, beans, config, tests) repeat in
every later service.

**Step 1 — You (manual, on the personal PC — full admin):**
- Clone `stockforge-project-context`, `stockforge-web` (and later `stockforge-api`) here:
  ```
  git clone https://github.com/Stock-Forge/stockforge-project-context.git
  git clone https://github.com/Stock-Forge/stockforge-web.git
  ```
- In org `Stock-Forge`, create empty repo `stockforge-api` (no README/.gitignore/license).
- Install **JDK 21** (Temurin 21: https://adoptium.net — installer is fine on this
  machine; verify with `java -version`). Maven is NOT needed — the scaffold uses the
  Maven Wrapper (`mvnw`), which pins its own version.
  > Note: the no-admin device uses Java 17 + no Maven — it can only do frontend/state
  > work; all JVM work happens here until containers (Phase 12-13).

**Step 2 — AI session does:**
1. Startup protocol (pull here, read context/state/prompts, reconcile).
2. Create local folder `C:\CODE\HFT Application\stockforge-api` — its OWN git repo.
3. Scaffold Spring Boot (Maven) with `spring initializr`:
   - explain `pom.xml`, `@RestController`, `application.yml`
   - `GET /api/health` endpoint with JSON `{status: "UP"}`
   - structured logging (JSON or key=value)
   - correlation ID filter (read/set `X-Correlation-Id`)
   - one unit test (e.g. MockMvc on `/api/health`)
4. Run tests + start app; verify `http://localhost:8080/api/health`.
5. Commit + push to `Stock-Forge/stockforge-api` (message describes the change).
6. **CENTRAL-STATE RULE:** update state HERE — `CURRENT_STATE.md` (Day 5 done → pointer to
   Day 6), `SESSION_PROMPTS.md` (Session 5 entry), `DAY_BY_DAY_GUIDE.md` (mark Day 5 done),
   `CHANGELOG.md`, `PROJECT_CONTEXT.md` if changed.
7. Commit + push this repo. Verify BOTH pushes.

**Expected result:** `stockforge-api` on GitHub; `GET /api/health` returns 200 with a
JSON body; a correlation ID appears in logs.

**Environment note:** Day 5 must run on a machine with JDK 21 (the personal PC, full
admin). No-admin device: Java 17 + no Maven → defer JVM work there.

---

## GIT STATUS (verify on the other device)

```
git status        # clean in both repos
git log --oneline # project-context → closeout commit; web → 8e7d075
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
`SESSION_PROMPTS.md`, `PROJECT_CONTEXT.md`, `DAY_BY_DAY_GUIDE.md`, `ISSUES_LOG.md`,
`CHANGELOG.md`. These files AUTO-UPDATE at the end of every session (the AI does it,
no reminder needed). We often work in a DIFFERENT repo that day (e.g. `stockforge-contracts`);
that repo gets only its own code, tests, and README. State updates happen HERE, always, and are
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
5. End of day (mandatory — auto-updates, no prompt needed):
   - run tests
   - update the working repo's own README
   - CENTRAL-STATE RULE: update PROJECT_CONTEXT.md (if architecture changed),
     CURRENT_STATE.md, SESSION_PROMPTS.md, DAY_BY_DAY_GUIDE.md, ISSUES_LOG.md,
     CHANGELOG.md HERE in stockforge-project-context — never in the working repo
   - git status + git diff review
   - git add + git commit (message describes the change, not the day)
   - git push in BOTH repos; verify (git status -sb up to date)
   - never claim saved until pushes are verified
```
