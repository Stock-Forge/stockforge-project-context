# CURRENT_STATE.md

> Current progress state. Updated after every meaningful session.
> Read this first (after PROJECT_CONTEXT.md) when starting a new AI session.
> This file always shows: where we are (day by day) + the prompt to begin the next day.

---

## Snapshot

| Field | Value |
|---|---|
| Date | 2026-08-08 |
| Phase | Phase 1 — Git/GitHub organization and repository foundation |
| Day | **Day 6 (DONE) — next is Day 7: `stockforge-auth` part 2 (roles + JWT verification)** |
| Status | Day 6 done & pushed: `stockforge-auth` built (Spring Boot 4.1.0) — bcrypt register + JWT login, 8 tests pass, curl verified 201/409/200/401/400. State updated & pushed here. |
| Current repository | stockforge-project-context (state repo) — Day 7 continues in `stockforge-auth` |
| Current branch | main |
| Current commit | stockforge-project-context: new closeout commit; stockforge-auth: `2306f08`; stockforge-api: `b0788bb`; stockforge-web: `8e7d075` |

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
| 6 | `stockforge-auth` — registration/login (bcrypt + JWT), roles | ✅ done (register/login) |
| 7 | `stockforge-auth` part 2 — roles + JWT verification (`@Authenticated`) reused by the API | ⏭️ **NEXT** |
| 8-9 | `stockforge-order-service` — order lifecycle (in-memory), risk mock | planned |
| 10+ | PostgreSQL → risk → matching → market-data → portfolio → notification → integration → … → HFT (roadmap §21) | planned |

---

## What Day 6 completed

- GitHub repo **`stockforge-auth`** created (empty) by the user; local repo created as its
  own git repo (sibling, never nested). First commit `1e3116d` pushed (tracking set).
- Scoped this build to the auth contract in `stockforge-contracts` (register → 201/400/409,
  login → 200/401) before writing code — contract-first.
- Spring Boot 4.1.0 scaffold (web + actuator + security + **validation**) via `start.spring.io`,
  Java 21 language level, Maven Wrapper. Reused the Day 5 pattern (health, structured logging).
- **bcrypt password hashing** — never stored in plain text (`BCryptPasswordEncoder`).
- **In-memory user store** for now (`UserStore`, thread-safe map); DB comes later.
- **`POST /api/auth/register`** → 201 + user (no password field); duplicate email → 409;
  short password (< 8) → 400 (jakarta validation).
- **`POST /api/auth/login`** → 200 + signed **JWT** (jjwt 0.13.0, HS256, subject = email, 1h
  expiry) + user; wrong password or unknown email → same 401 (no user-probing).
- **JwtService** (sign + parse, tampered tokens rejected) and **SecurityConfig** (stateless,
  no CSRF/form login; only `/api/auth/*` + health permitted; the rest stays locked — the
  JWT filter that unlocks it comes Day 7).
- Tests: MockMvc (register 201, duplicate 409, login 200 + JWT, wrong password 401, short
  password 400) + JwtService unit tests. **All 8 pass.**
- Ran the app (`.\mvnw spring-boot:run`), verified with curl the full matrix
  **201 → 409 → 200 → 401 → 400**; stopped the server. The user manually re-verified with
  curl (same matrix) while we continued.
- **403 error-masking bug found + fixed (`2306f08`):** manual curl testing showed 403 on
  register. Root cause: (1) PowerShell→curl mangled inline JSON bodies (`{email:...}` → parse
  error), and (2) the secured `/error` dispatch turned every real error (400/404) into 403.
  Fix: `.requestMatchers("/error").permitAll()`; added `test-auth.ps1` (file-based bodies,
  immune to the quoting bug). Full matrix re-verified 201/409/200/401/400; 8 tests pass.
- Day 6 issues logged in `ISSUES_LOG.md` (missing `spring-boot-starter-validation`;
  test-isolation bug — shared in-memory store across tests; leftover Day 5 process on :8080;
  403 error-masking; PathPatternRequestMatcher false lead).
- **NEW WORKFLOW RULE (baked into `START_OF_DAY.md`):** the AI must deliver the full
  day briefing BEFORE any code and WAIT for acknowledgment — the user tests manually
  alongside, so the plan comes first every day.

## What is deliberately NOT done

- **Day 7 (next):** roles + a `JwtAuthenticationFilter` (populates Spring SecurityContext
  from the bearer token) + a protected `/api/auth/me` endpoint, so `stockforge-api` can
  later reuse the same verification. Not yet started.
- **Device B clone still unverified** (user-owned) — clone this repo on the other device:
  ```
  git clone https://github.com/Stock-Forge/stockforge-auth.git
  ```
- Auth service has no persistence (in-memory users) and no logout/refresh/revocation yet
  (expiry exists via JWT exp). Tokens are single-service for now — cross-service reuse is
  Day 7.
- The API still talks to nothing: no routing to auth, no JWT-protected business endpoints.
- No CI yet (Phase 14); no rate limiting (a Day 15+ concern).

## Incomplete work (open items carried forward)

```
- Device B clone verification (user-owned, do anytime).
- Contract tooling + contract tests (deferred to service phases / Phase 14).
- stockforge-web is a static shell with no API wiring; stockforge-api has no real endpoints.
- stockforge-auth: no persistence (in-memory users), no logout/refresh/revocation.
- Cross-device JVM note: this machine has JDK 26; if the personal PC has JDK 21, both are fine
  (java.version=21 targets 21+). Maven comes from the wrapper everywhere.
```

---

## NEXT DAY PROMPT — DAY 7: `stockforge-auth` part 2 (roles + JWT verification)

**How to start:** copy-paste the FULL content of
`stockforge-project-context\project-context\START_OF_DAY.md` into the new AI session.
Below is the day-specific plan the AI must follow.

### Day 7 — `stockforge-auth`: roles + `@Authenticated` (JWT-protected endpoints)

**Goal:** make the JWT actually *protect* something. Add a **`JwtAuthenticationFilter`**
that reads the `Authorization: Bearer <token>` header, verifies it, and populates the
Spring **SecurityContext**; add a protected **`GET /api/auth/me`** endpoint returning the
current user from the token; and exercise it with roles (`USER`, `ADMIN`). This is the
mechanism `stockforge-api` will reuse on Day 8+ so every business endpoint can trust the
token without re-verifying it per service by hand.

**Why / production thinking:** a token you can verify but never use to protect anything is
useless. Real platforms put an auth filter/gateway in front of every protected endpoint;
the filter verifies the JWT ONCE per request and lets the endpoint read the identity from
context. Least privilege + role checks are how trading platforms stop a user seeing
another user's orders/positions.

**Step 1 — You (manual):** nothing to create — `stockforge-auth` exists. (Optional: create
empty repo `stockforge-order-service` if we finish early and time allows.)

**Step 2 — AI session does:**
1. Startup protocol (pull both repos, read context/state/prompts, reconcile).
2. **Briefing first, then WAIT for acknowledgment (EXPLAIN-FIRST RULE).**
3. Add `JwtAuthenticationFilter` (a `OncePerRequestFilter`, `@Order(2)` — after the
   correlation-ID filter): parse the Bearer token via `JwtService`, build a Spring
   `Authentication` from the subject + roles, set the SecurityContext; on failure leave
   the context unauthenticated (don't throw — endpoints decide).
4. Move role membership where the filter can read it: `JwtService` gains role claims
   (e.g. `roles=USER,ADMIN`) OR the filter loads the user from `UserStore` by email.
5. Add `@Authenticated` annotation + a check that only lets authenticated requests through
   on chosen endpoints; protect `GET /api/auth/me` (returns `UserResponse` for the token's
   subject). Demonstrate a role gate (e.g. an `ADMIN`-only path or `@PreAuthorize("hasRole('ADMIN')")`).
6. Tests: `/api/auth/me` with valid token → 200 + the right user; missing/invalid/expired
   token → 401; role-protected path allows ADMIN, denies USER. Keep all existing tests green.
7. Run tests; verify with curl (login → take JWT → call `/api/auth/me` with and without
   the header).
8. Commit + push to `Stock-Forge/stockforge-auth` (message describes the change).
9. **CENTRAL-STATE RULE:** update state HERE — `CURRENT_STATE.md` (Day 7 done → Day 8),
   `SESSION_PROMPTS.md` (Session 7 entry), `DAY_BY_DAY_GUIDE.md` (mark Day 7 done),
   `ISSUES_LOG.md`, `LEARNING_LOG.md`, `JOURNEY_SO_FAR.md` (teach-back), `CHANGELOG.md`,
   `PROJECT_CONTEXT.md` (auth status if changed).
10. Commit + push this repo. Verify BOTH pushes.

**Expected result:** `stockforge-auth` on GitHub has a protected `/api/auth/me`; a valid
JWT returns the current user, an invalid/missing one gets 401; role gate demonstrable;
tests pass; Day 8 starts `stockforge-order-service`.

**Environment note:** JDK 21+ required (this device has JDK 26 — fine). Maven comes from
the wrapper. Keep the stateless security setup; do NOT add sessions or form login.

---

## GIT STATUS (verify on the other device)

```
git status        # clean in all repos
git log --oneline # project-context → closeout commit; auth → 2306f08; api → b0788bb; web → 8e7d075
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
`LEARNING_LOG.md`, `CHANGELOG.md`. These files AUTO-UPDATE at the end of every session (the AI does it,
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
      LEARNING_LOG.md, CHANGELOG.md HERE in stockforge-project-context — never in the working repo
   - git status + git diff review
   - git add + git commit (message describes the change, not the day)
   - git push in BOTH repos; verify (git status -sb up to date)
   - never claim saved until pushes are verified
```
