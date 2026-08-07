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
| Day | **Day 5 (DONE) — next is Day 6: `stockforge-auth`** |
| Status | Day 5 done & pushed: `stockforge-api` scaffolded (Spring Boot 4.1.0), `/api/health` verified 200, tests pass. Tooling gate RESOLVED — this device now has JDK 26 (stack wants 21+, OK). State updated & pushed here. |
| Current repository | stockforge-project-context (state repo) — Day 6 builds in `stockforge-auth` |
| Current branch | main |
| Current commit | stockforge-project-context: new closeout commit; stockforge-api: `b0788bb`; stockforge-web: `8e7d075` |

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
| 5 | `stockforge-api` — Spring Boot gateway scaffold | ✅ done |
| 6 | `stockforge-auth` — registration/login (bcrypt + JWT), roles | ⏭️ **NEXT** |
| 7 | `stockforge-auth` part 2 — roles + `@Authenticated` check used by the API | planned |
| 8-9 | `stockforge-order-service` — order lifecycle (in-memory), risk mock | planned |
| 10+ | PostgreSQL → risk → matching → market-data → portfolio → notification → integration → … → HFT (roadmap §21) | planned |

---

## What Day 5 completed

- GitHub repo **`stockforge-api`** created (empty) by the user; local repo created as its own git repo (sibling, never nested).
- **Environment change resolved the tooling gate:** this device now has **JDK 26** (`java -version` = 26; stack wants 21+, satisfied) and no Maven — fine, the Maven Wrapper (`mvnw`, pinned to Maven 3.9.16) handled the build.
- Scaffolded **Spring Boot 4.1.0** (Maven) via `start.spring.io` (`webmvc` + `actuator` deps, Java 21 language level). Explained `pom.xml` (parent, deps, wrapper), `@RestController`, `application.yml`.
- **`GET /api/health`** → `{"status":"UP"}` (200).
- **Structured logging** — key=value pattern in `application.yml`; every line carries `[correlationId=...]`.
- **Correlation-ID filter** (`CorrelationIdFilter`, `@Order(1)`): reads `X-Correlation-Id` or generates a UUID, stores in MDC, echoes on the response.
- Tests: MockMvc on `/api/health` (200 + JSON) + 2 filter unit tests (preserve incoming / generate when absent). **All 4 tests pass.**
- Ran the app (`./mvnw spring-boot:run`), verified `curl http://localhost:8080/api/health` → 200, and `X-Correlation-Id` header round-trip; stopped the server.
- Committed `3374c38` (+ `b0788bb` removing a stray `run.log`) and pushed to `Stock-Forge/stockforge-api` (tracking set).

**Stack update (recorded in ADR 0002 + PROJECT_CONTEXT §5):** Spring Boot **3.x → 4.1.0** — `start.spring.io` no longer generates 3.x scaffolds. Verified running on Java 26.

## What is deliberately NOT done

- **Device B clone still unverified** (user-owned) — clone this repo on the other device:
  ```
  git clone https://github.com/Stock-Forge/stockforge-api.git
  ```
- The API still talks to nothing: no auth, no persistence, no downstream services, no routing to stockforge-web yet.
- No CI yet (Phase 14); actuator exposes only the health endpoint (no other metrics yet).
- `stockforge-web` still makes zero HTTP calls — wiring it to `stockforge-api` comes after the API grows.

## Incomplete work (open items carried forward)

```
- Device B clone verification (user-owned, do anytime).
- Contract tooling + contract tests (deferred to service phases / Phase 14).
- stockforge-web is a static shell with no API wiring; stockforge-api has no real endpoints yet.
- Cross-device JVM note: this machine has JDK 26; if the personal PC has JDK 21, both are fine
  (java.version=21 targets 21+). Maven comes from the wrapper everywhere.
```

---

## NEXT DAY PROMPT — DAY 6: `stockforge-auth`

**How to start:** copy-paste the FULL content of
`stockforge-project-context\project-context\START_OF_DAY.md` into the new AI session.
Below is the day-specific plan the AI must follow.

### Day 6 — `stockforge-auth`: registration & login

**Goal:** a Spring Boot auth service with `POST /api/auth/register` and
`POST /api/auth/login` using **bcrypt** password hashing and a **JWT**; wrong
password → 401.

**Why / production thinking:** security foundation. Passwords are never stored in plain
text; we issue signed tokens the API verifies without a DB lookup per request. This is
the highest-risk area of any trading platform — identity is where real firms get fined.

**Step 1 — You (manual):**
- In org `Stock-Forge`, create empty repo `stockforge-auth` (no README/.gitignore/license).

**Step 2 — AI session does:**
1. Startup protocol (pull here, read context/state/prompts, reconcile).
2. Create local folder `C:\CODE\HFT Application\stockforge-auth` — its OWN git repo.
3. Scaffold Spring Boot (same as Day 5: `start.spring.io`, webmvc + actuator + a new
   `security` dependency for bcrypt), reuse the Day 5 pattern (health, correlation ID).
4. Implement `POST /api/auth/register` + `POST /api/auth/login`:
   - users held **in memory** for now (DB comes later) — bcrypt hash the password
   - issue a signed JWT on successful login; 401 on wrong credentials
5. Tests: register then login returns a token; wrong password = 401; duplicate
   register handled.
6. Run tests; verify with `curl` (register → login → JWT in response).
7. Commit + push to `Stock-Forge/stockforge-auth` (message describes the change).
8. **CENTRAL-STATE RULE:** update state HERE — `CURRENT_STATE.md` (Day 6 done → Day 7),
   `SESSION_PROMPTS.md` (Session 6 entry), `DAY_BY_DAY_GUIDE.md` (mark Day 6 done),
   `ISSUES_LOG.md`, `CHANGELOG.md`, `PROJECT_CONTEXT.md` if changed.
9. Commit + push this repo. Verify BOTH pushes.

**Expected result:** `stockforge-auth` on GitHub; register → login returns a JWT;
wrong password = 401; tests pass.

**Environment note:** JDK 21+ required (this device has JDK 26 — fine). Maven comes
from the wrapper. If `spring-boot-starter-security` pulls in a login screen you didn't
ask for, we restrict it to the auth endpoints only (that's a Day 6 teaching point).

---

## GIT STATUS (verify on the other device)

```
git status        # clean in both repos
git log --oneline # project-context → closeout commit; api → b0788bb; web → 8e7d075
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
