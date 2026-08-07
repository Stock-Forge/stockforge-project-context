# StockForge — Changelog

All notable changes per project day. Format: date — Day N — summary.

## 2026-08-08 — Day 6 (tooling: Postman collections)

- Added `testing/*.postman_collection.json` to `stockforge-auth` (`43dba23`) and
  `stockforge-api` (`acb07a0`) so the app can be tested from Postman instead of curl.
  Auth collection: health + register 201 + duplicate 409 + short password 400 + login 200
  (auto-saves JWT to `accessToken` variable) + wrong password 401, with assertions.

- `stockforge-auth` `2306f08`: fixed **403 error-masking** — permitted `/error` dispatch so
  real errors (400/404) keep their status instead of becoming 403 for anonymous callers;
  added `test-auth.ps1` (file-based curl bodies, immune to PowerShell quoting). Root cause of
  the manual-test 403s was mangled inline JSON from PowerShell→curl, not the security rules.
- Case studies logged in `ISSUES_LOG.md` (mangled curl body + secured /error; the
  PathPatternRequestMatcher false lead).

- `stockforge-auth` repo created and pushed (`1e3116d`): Spring Boot 4.1.0 auth service —
  `POST /api/auth/register` (bcrypt hash, 201/409/400) + `POST /api/auth/login`
  (200 + signed JWT via jjwt 0.13.0, wrong password → 401), in-memory `UserStore`,
  stateless `SecurityConfig`. 8 tests pass; curl-verified 201→409→200→401→400.
- New workflow rule baked into `START_OF_DAY.md`: **explain-first** — the AI delivers the
  full day briefing and waits for acknowledgment before any code/commands (user tests
  manually alongside).
- Day 6 issues logged in `ISSUES_LOG.md` (missing `spring-boot-starter-validation`,
  shared-store test-isolation bug, leftover Day 5 process on :8080, Boot 4 `web` id).

## 2026-08-07 — Day 5 closeout (additions)

- New `project-context/LEARNING_LOG.md`: day-wise study notes (what we built / concept
  in one sentence / what I should remember / production-HFT relevance) covering Days 0-5.
  Baked into `START_OF_DAY.md` Phase 3 as a mandatory end-of-day step so every future
  day is logged; referenced in the central-state file lists (SESSION_PROMPTS,
  DAY_BY_DAY_GUIDE, CURRENT_STATE).

## 2026-08-07 — Day 5

- `stockforge-api` repo created and pushed (`3374c38` + `b0788bb`): Spring Boot **4.1.0**
  scaffold (Maven Wrapper, Java 21 language level) with `GET /api/health`
  (→ `{"status":"UP"}`), key=value structured logging, and a correlation-ID filter
  (MDC + `X-Correlation-Id` echo). 4 tests pass; endpoint verified with curl.
- Stack update: Spring Boot **3.x → 4.1.0** (start.spring.io is 4.x-only) — recorded in
  ADR 0002 and PROJECT_CONTEXT §5. Tooling gate resolved: this device now has JDK 26.
- Day 5 issues logged in `ISSUES_LOG.md` (parent-POM version label vs Maven Central,
  Boot 4 `@AutoConfigureMockMvc` relocation, accidental `run.log` commit).

## 2026-08-07 — Day 5 (partial, blocked at tooling gate) — 0.1% habits added

- Added the **10 "0.1%" engineering practices** to the project (`PROJECT_CONTEXT.md`
  §25 + §26 primitive side quests): teach-back, hypothesis-before-change, deep-not-wide,
  build-it-wrong, build-the-primitive, read-real-platforms, bugs-as-case-studies,
  observability-from-day-one, teach-someone, monthly code review.
- Dedicated **Days 40-43** in the roadmap + guide: platform study (Zerodha/Groww),
  SRE-style incident post-mortem, teach someone else, monthly code review.
- Baked habits into `START_OF_DAY.md` + `SESSION_PROMPTS.md`; upgraded `ISSUES_LOG.md`
  template to case-study format (symptom → detection → cause → fix → prevention → production).

## 2026-08-07 — Day 5 (partial, blocked at tooling gate)

- Started Day 5 (`stockforge-api`) startup protocol; this device has Java 17 + no Maven +
  no admin → JDK 21 setup and the build move to the personal PC (full admin).
- Prompt upgrade: sessions now begin with a full-detail briefing (steps + tech stack +
  Zerodha/Groww production comparison).
- New `project-context/ISSUES_LOG.md` (per-day issues: symptom/cause/fix/production),
  auto-updated at end of every session alongside the other state files.

## 2026-08-07 — Day 4

- `stockforge-web` repo created and pushed (`8e7d075`): Vite + React 19 + TypeScript
  scaffold with login placeholder + dashboard shell ("API not connected yet"). Lint,
  build, and dev server (localhost:5173) verified.

## 2026-08-06 — Day 3

- `stockforge-contracts` repo created and pushed: OpenAPI v1.0.0 (10 paths, 14 schemas)
  + 7 Kafka event contracts + README. Contract-first established.

## 2026-08-06 — Day 2

- GitHub org `Stock-Forge` created (project/product name remains `StockForge`).
- Repository `stockforge-project-context` created and `main` pushed to GitHub.
- Org name + roadmap updates documented in `PROJECT_CONTEXT.md`.
- Expanded the 30-minute learning roadmap to Day 39+ (HFT evolution).

## 2026-08-06 — Day 1

- Phase 0 architecture approved by user (stack: Java 21 + Spring Boot, React+TS,
  PostgreSQL, Redis, Kafka, Docker, kind→EKS, GitHub Actions + Jenkins, k6, Prometheus/Grafana).
- ADR 0001 (org/repo layout + two-device workflow) and ADR 0002 (technology stack) accepted.
- `DAY_BY_DAY_GUIDE.md` created (GitHub setup, folder structure, production thinking).

## 2026-08-06 — Day 0

- Project foundation: `MASTER_PROMPT.md`, `PROJECT_CONTEXT.md`, `CURRENT_STATE.md`,
  `SESSION_PROMPTS.md`, `adr/` folder, `README.md`.
- Initial commit `b9e8cb3`.
