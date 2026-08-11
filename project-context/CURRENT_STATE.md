# CURRENT_STATE.md

> Current progress state. Updated after every meaningful session.
> Read this first (after PROJECT_CONTEXT.md) when starting a new AI session.
> This file always shows: where we are (day by day) + the prompt to begin the next day.

---

## Snapshot

| Field | Value |
|---|---|---|
| Date | 2026-08-11 |
| Phase | Phase 1 — Git/GitHub organization and repository foundation |
| Day | **Day 7 COMPLETE — next is Day 8: `stockforge-order-service` (order lifecycle, in-memory)** |
| Status | Day 7 (`JwtAuthenticationFilter` + roles in JWT + `GET /api/auth/me` + ADMIN role gate) done & pushed (`3255bab`). JWT now actually protects endpoints: valid token → 200, missing/invalid/expired → 401, USER on ADMIN path → 403. Real bug found + fixed: anonymous requests returned 403 not 401 (no AuthenticationEntryPoint after disabling httpBasic/formLogin). 19 tests green. Live curl matrix verified USER + ADMIN paths. Day 8 starts `stockforge-order-service` per `DAY_BY_DAY_GUIDE.md`. |
| Current repository | stockforge-project-context (state repo) — Day 8 continues in a NEW repo `stockforge-order-service` |
| Current commit | stockforge-project-context: Day 7 closeout (this commit); stockforge-auth: `3255bab`; stockforge-api: `acb07a0`; stockforge-web: `8e7d075`; stockforge-contracts: `e1d65cb` |

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
| 7 | `stockforge-auth` part 2 — roles + JWT verification (`JwtAuthenticationFilter` + `/api/auth/me` + role gate) | ✅ done |
| 8-9 | `stockforge-order-service` — order lifecycle (in-memory), risk mock | ⏭️ **NEXT** |
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

## What Day 7 completed

- **`JwtService` gained role claims:** `generateToken(subject, roles)` now embeds
  `roles` in the token; `parseClaims` returns a `JwtClaims(subject, roles)` record;
  `parseSubject` delegates to it. Roles ride inside the JWT (self-contained) so
  `stockforge-api` can reuse the same verification from Day 8 without touching `UserStore`.
- **`JwtAuthenticationFilter`** (a `OncePerRequestFilter`): reads `Authorization: Bearer`,
  verifies via `JwtService`, builds a Spring `UsernamePasswordAuthenticationToken`
  (principal = email, authorities = `ROLE_`-prefixed roles) and sets the SecurityContext.
  On ANY verification failure it clears the context and does NOT throw — the security
  machinery answers 401.
- **`SecurityConfig`**: `@EnableMethodSecurity` (turns on `@PreAuthorize`), the JWT filter
  inserted before `UsernamePasswordAuthenticationFilter`, stateless, CSRF off,
  httpBasic/formLogin disabled.
- **401/403 bug found + fixed (the day's sharp edge):** anonymous requests to protected
  endpoints returned **403, not 401**, because disabling httpBasic/formLogin removed the
  default `AuthenticationEntryPoint` (Spring falls back to `Http403ForbiddenEntryPoint`).
  Fix: explicit `authenticationEntryPoint` that answers 401. Tests caught it; curl confirmed
  the fix live. This is exactly the authn (401) vs authz (403) split from the day's concepts.
- **`GET /api/auth/me`** — reads identity from the SecurityContext (never from the request
  body/headers) and returns the `UserResponse`; missing/invalid/expired token → 401.
- **`GET /api/auth/admin/ping`** with `@PreAuthorize("hasRole('ADMIN')")` — role gate demo:
  ADMIN token → 200, USER token → 403, anonymous → 401.
- **`AdminBootstrap`** `CommandLineRunner`: seeds `admin@stockforge.dev` (ADMIN) from
  `auth.admin.email/password` in `application.yml`, idempotent (never overwrites an
  existing user). Passwords are still bcrypt-hashed.
- **Tests: 19 pass** (12 controller incl. 7 new Day 7 cases, 4 JWT incl. roles, 2 bootstrap,
  1 context). Live curl matrix verified end-to-end: USER path (me 200/401/401, admin/ping
  403) and ADMIN path (login → me 200 + roles[ADMIN] → admin/ping 200 "ADMIN access granted").
- A stale Day 6 server process was still holding :8080 — stopped it, restarted the new build.

## What is deliberately NOT done

- **Day 8 (next):** `stockforge-order-service` — order lifecycle (NEW → ACCEPTED → FILLED /
  CANCELLED) in-memory; mocked risk check comes Day 9. Repo does not exist yet (user creates it).
- **Device B clone still unverified** (user-owned) — clone this repo on the other device:
  ```
  git clone https://github.com/Stock-Forge/stockforge-auth.git
  ```
- Auth service has no persistence (in-memory users) and no logout/refresh/revocation yet.
  Roles take effect on the next login — an already-issued JWT keeps its roles until expiry
  (that's why prod uses short-lived access tokens + refresh tokens).
- `stockforge-api` still does NOT reuse the JWT verification yet (that's Day 8+): the
  filter lives in `stockforge-auth`; cross-service token trust is the next step.
- No CI yet (Phase 14); no rate limiting (a Day 15+ concern).

## Incomplete work (open items carried forward)

```
- Device B clone verification (user-owned, do anytime).
- Contract tooling + contract tests (deferred to service phases / Phase 14).
- stockforge-web is a static shell with no API wiring; stockforge-api has no real endpoints.
- stockforge-auth: no persistence (in-memory users), no logout/refresh/revocation yet; roles
  apply at next login (issued JWTs keep their roles until expiry).
- stockforge-api does not yet reuse the auth JWT verification (Day 8+ step).
- Cross-device JVM note: this machine has JDK 26; if the personal PC has JDK 21, both are fine
  (java.version=21 targets 21+). Maven comes from the wrapper everywhere.
- JFR PRACTICAL SIDE QUEST (2026-08-10, ADR 0003): small plain-Java app with a controlled
  allocation/lock-contention problem → record with JFR → inspect GC/CPU/lock behavior. This is
  the first practical step of the Core Trading Stack decision (Java-first trading engine).
```

---

## Session note — 2026-08-10 (between Day 6 and Day 7)

- Deep-study session (vault Tech Stack note → LEARNING_LOG): **GC, JFR, lock contention**, Spring
  Boot's role, and the **Core Trading Stack decision**.
- **ADR 0003 PROPOSED** — hot path (matching, market data, strategy, risk, OMS, gateway) = **plain
  Java**, not Spring Boot; control plane stays on Spring Boot; C++ comparison later.
- New cheat sheet: `project-context/cheatsheets/hft-performance-cheat-sheet.html` + vault
  `TechStack/JVM Performance Cheat Sheet` + vault notes GC/JFR/Lock Contention/Core Trading Stack.
- Day 7 (auth roles + `@Authenticated`) is unchanged as the next coding day.

---

## Session note — 2026-08-11 (Deep Review Ritual 1 closeout + frontend study)

- **Deep Review Ritual 1 COMPLETED:** walked the full Days 0–6 review one-on-one (architecture,
  each day's concepts, the auth code walkthrough, every issue as a mini case study, the run/test
  playbook, production mapping). User's homework answers all correct — highlights:
  - Type-check-before-bundle = the bundler isn't the contract prover; cheap checks upstream.
  - Re-render cascade and lock-wait are the SAME smell: resources consumed without useful progress.
  - Poll ≠ push on the hot path: a poll can only observe at its own cadence → quantized latency.
    HFT reacts by event/heartbeat, not ask.
  - `test-auth.ps1` file bodies = bug workaround (PowerShell→curl strips quotes) + good hygiene.
  - JWT is client-side ID (safe to return); password hash is server secret (never expose).
- **Cheat-sheet teaching style is now a PROJECT_CONTEXT key rule** (user preference): one-picture
  diagram first → key findings in plain words → map unfamiliar→familiar → plant-bug-then-hunt lab.
- New frontend cheat sheet (React/TS/Vite) — the frontend twin of the JVM cheat sheet; frontend
  debugging lab added to SideQuests. Vault mirrors created.
- **Day 7 is next** (auth roles + `JwtAuthenticationFilter` + `/api/auth/me` + role gate).

---

## Session note — 2026-08-11 (Day 7 — auth roles + JWT verification, DONE)

- **Day 7 executed and closed out:** `JwtService` role claims → `JwtAuthenticationFilter`
  → `@EnableMethodSecurity` + `@PreAuthorize("hasRole('ADMIN')")` → `GET /api/auth/me` →
  `GET /api/auth/admin/ping` → `AdminBootstrap` seed. Committed `3255bab`, pushed.
- **The day's sharp edge — 401 vs 403:** tests failed because anonymous requests got 403,
  not 401. Root cause: with httpBasic/formLogin disabled, Spring Security has no default
  `AuthenticationEntryPoint` and falls back to `Http403ForbiddenEntryPoint`. Fix: explicit
  entry point → 401 for unauthenticated, 403 for authenticated-but-denied. Verified live.
- **Manual-testing lessons re-confirmed live:** PowerShell→curl mangled inline JSON bodies
  again (file bodies fixed it — the Day 0-6 lesson holds); a stale Day 6 server was still
  on :8080 (stopped it; this is the 2nd time this exact issue recurred — see ISSUES_LOG).
- **19 tests green** (12 controller, 4 JWT, 2 bootstrap, 1 context). Full curl matrix
  verified: me 200/401/401, admin/ping 403 (USER) + 200 (ADMIN).
- **Day 8 next:** `stockforge-order-service` — user creates the empty GitHub repo; AI
  scaffolds Spring Boot 4.1.0, in-memory order store, lifecycle state machine, tests.

---

## NEXT DAY PROMPT — DAY 8: `stockforge-order-service` (orders, in-memory first)

**How to start:** copy-paste the FULL content of
`stockforge-project-context\project-context\START_OF_DAY.md` into the new AI session.
Below is the day-specific plan the AI must follow.

### Day 8 — `stockforge-order-service`: orders with a lifecycle (in-memory)

**Goal:** a new service for the core trading domain — create/list/cancel orders with an
explicit state machine (**NEW → ACCEPTED → FILLED / CANCELLED**), stored in memory,
with REST endpoints and tests. The mock risk check is Day 9. This is where the trading
business logic starts living.

**Why / production thinking:** orders are the heart of a trading platform and every
state transition matters — this is where firms get fined for bugs. We keep it in memory
first so the 30-minute unit stays small; PostgreSQL comes at Day 10.

**Step 1 — You (manual):** create empty repo **`stockforge-order-service`** on GitHub
(`Stock-Forge/stockforge-order-service`), no README/license (we add them locally).

**Step 2 — AI session does:**
1. Startup protocol (pull both repos, read context/state/prompts, reconcile).
2. **Briefing first, then WAIT for acknowledgment (EXPLAIN-FIRST RULE).**
3. Local repo as a sibling (never nested); scaffold Spring Boot 4.1.0 (web + actuator +
   validation) via `start.spring.io` with Maven Wrapper, Java 21 language level; reuse the
   Day 5/6 pattern (health, structured logging, correlation ID, `test-auth`-style file
   bodies for curl). Port **8081** (8080 is auth).
4. Domain: `Order` (id, clientOrderId, symbol, side, quantity, price, status, timestamps),
   an in-memory thread-safe `OrderStore`, an `OrderService` with explicit state transitions
   (invalid transitions rejected), `OrderController` (create/list/get/cancel).
5. Contract-first: define the order endpoints in the OpenAPI before coding, per the
   `stockforge-contracts` contract.
6. Tests: create → ACCEPTED, cancel → CANCELLED, FILLED via a simulated fill (Day 9 risk
   gate later), invalid transition rejected, not-found 404, validation 400.
7. Run tests; verify with curl (create/list/cancel).
8. Commit + push to `Stock-Forge/stockforge-order-service`.
9. **CENTRAL-STATE RULE:** update state HERE — `CURRENT_STATE.md` (Day 8 done → Day 9),
   `SESSION_PROMPTS.md` (Session 8 entry), `DAY_BY_DAY_GUIDE.md`, `ISSUES_LOG.md`,
   `LEARNING_LOG.md`, `JOURNEY_SO_FAR.md`, `CHANGELOG.md`, `PROJECT_CONTEXT.md` (services map).
10. Commit + push this repo. Verify BOTH pushes.

**Expected result:** `stockforge-order-service` on GitHub with a working order lifecycle
state machine, REST create/list/get/cancel, unit tests green; a real order survives its
transitions and invalid ones are rejected.

**Environment note:** JDK 21+ (this device has JDK 26 — fine). Maven comes from the wrapper.
Same Spring Boot 4.1.0 as auth/api — reuse their pom pattern.

---

## GIT STATUS (verify on the other device)

```
git status        # clean in all repos
git log --oneline # project-context → Day 7 closeout; auth → 3255bab; api → acb07a0; web → 8e7d075
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
