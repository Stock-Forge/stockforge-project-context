# SESSION_PROMPTS.md

> Reusable prompts for future sessions. After every session, update this file with
> a new continuation prompt and keep the **NEW SESSION RESUME PROMPT** up to date.
> These prompts are version-controlled project artifacts.

---

## NEW SESSION RESUME PROMPT (copy-paste this into a completely new AI session)

> **Every day:** use `project-context/START_OF_DAY.md` — it is the canonical daily-start
> prompt (reconstruct → work one 30-minute day → mandatory end-of-day sequence).
> The prompt below is the same protocol in compact form.

```
STOCKFORGE — NEW SESSION RESUME PROMPT

You are starting a completely new session for the StockForge project.
You have ZERO conversation history. Do NOT assume any previous context.

Step 1 — Sync:
Run: git pull
Then inspect: git status, git branch, git log -5

Step 2 — Read project context:
Read: stockforge-project-context/project-context/PROJECT_CONTEXT.md

Step 3 — Read current state:
Read: stockforge-project-context/project-context/CURRENT_STATE.md

Step 4 — Read the latest continuation prompt:
Read: stockforge-project-context/project-context/SESSION_PROMPTS.md
(Use the most recent SESSION CONTINUATION PROMPT below; ignore superseded ones.)

Step 5 — Inspect the repository currently being worked on (per CURRENT_STATE.md).

Step 6 — Reconcile: compare saved state + prompt + actual repository state.
If they disagree, investigate BEFORE writing any code.

Step 7 — Report this exact block, then continue only with the next 30-minute task:

Project reconstructed.

Current phase:
Current day:
Current repository:
Current branch:
Last completed task:
Current incomplete task:
What I understand:
Next 30-minute task:

Rules for this session:
- Begin by briefing the user in FULL detail, before any code or commands: WHAT we are
  doing today and HOW it helps the project; every step in order with exact commands/files;
  which technology from our stack is used at each step; and how this compares to a
  production-grade trading platform like Zerodha or Groww.
- State a hypothesis before every change ("I believe X because Y"), then measure and
  confirm. Go deep on today's topic (§25 habit 2-3).
- Teach before automating; explain WHAT/WHY/HOW/PRODUCTION.
- End of day: update `ISSUES_LOG.md` (every issue as a mini case study: symptom,
  detection, cause, fix, prevention, production relevance), write the teach-back
  paragraph into `JOURNEY_SO_FAR.md` (§25 habits 1, 7), and add a plain-language day
  entry to `LEARNING_LOG.md` (the user's re-readable study notes).
- One day = ~30 minutes. Prefer clean stopping points. Record incomplete work openly.
- Both GitHub Actions AND Jenkins are first-class CI/CD. Do not treat Jenkins as an afterthought.
- The project is production-style AND evolves toward HFT later. Never claim the initial
  microservice architecture is exchange-grade HFT.
- End of session: stop clean → run tests → update README → update PROJECT_CONTEXT.md
  (if architecture changed) → update CURRENT_STATE.md → update SESSION_PROMPTS.md →
  update LEARNING_LOG.md → git status → git diff review → git add → git commit →
  git push → verify push.
- Never claim the session is safely saved until git push is verified.
```

---

## SESSION CONTINUATION PROMPT TEMPLATE

After every session, append a filled copy of this template to `SESSION_PROMPTS.md`
(keep the most recent one at the top of the "Session history" section).

```
### Session N — <short summary> — <date>

Project:        StockForge
Current phase:  <phase>
Current day:    <Day N>
Current repo:   <repository>
Current branch: <branch>
Previous commit:<hash>

What was implemented:
- ...

What was learned:
- ...

Current problem / open questions:
- ...

Incomplete work (record exactly; next session continues from here):
- ...

Exact next task:
- ...

Commands to run:
- ...

Files to inspect:
- ...

Expected result:
- ...

Long-term direction:
- ...

Git verification: committed? (yes/no)  pushed? (yes/no)  verified? (yes/no)
```

---

## SESSION FORMAT (for each working day)

**Day N — Topic**

Goal: one clear 30-minute objective.

```
0–5 min     Concept
5–10 min    Production explanation
10–25 min   Implementation/testing
25–30 min   Review/documentation
```

End-of-session report:

- Completed today
- What I learned
- Production lesson
- Problems encountered
- Incomplete work
- Next 30-minute task
- Git status / commit / push status

---

## Review Session — Deep Review Ritual 1 (Days 0–6) + frontend study (2026-08-11)

Project:        StockForge
Current phase:  Phase 1 — Git/GitHub organization and repository foundation
Current day:    Day 0-6 review (COMPLETE) — Day 7 next
Current repo:   stockforge-project-context (state repo); Day 7 continues in `stockforge-auth`
Current branch: main
Previous commit: stockforge-project-context `7e3d77d`; auth `6be237e`; api `acb07a0`;
                 web `8e7d075`; contracts `e1d65cb`

What was done (review, not build):

- Completed the one-on-one walkthrough of `reviews/day-0-6-concept-review.html` (Ritual 1):
  big picture → stack → architecture → each day 0–6 → auth code walkthrough → issues as mini
  case studies → production map → run/test playbook. User answered every homework question
  correctly (see "What was learned").
- **Cheat-sheet teaching style promoted to a project rule** (PROJECT_CONTEXT §23 Key Rules):
  one-picture diagram first → key findings in plain words → map unfamiliar→familiar → a
  plant-the-bug-then-hunt lab. Vault Home conventions updated too.
- New `project-context/cheatsheets/frontend-cheat-sheet.md` (React/TS/Vite) — frontend twin of
  the JVM cheat sheet, incl. a frontend debugging lab (re-render storm / main-thread block /
  bundle bloat, hunted with React DevTools Profiler + Lighthouse). Vault mirror +
  `TechStack/React Vite TypeScript` link + SideQuests row added.
- No application code changed during this session.

What was learned (user's correct homework answers):

- Type-check before bundle: the bundler is not the contract prover; cheap checks run upstream.
- A child re-render when nothing changed = lock-wait = the same smell: work without useful
  progress (performance found in both stacks).
- Poll vs push on the hot path: a poll observes only at its own cadence → quantized latency;
  HFT reacts by event/heartbeat, never ask (the deeper perf reason behind the Day 5 bonus topic).
- File-based curl bodies = bug workaround for PowerShell→curl quote stripping + good hygiene.
- JWT is client-facing ID (safe to return); a password hash is server secret (never expose).

Problems encountered: none blocking (all review + doc work).

Incomplete work:

- Day 7 (next): `JwtAuthenticationFilter` (SecurityContext from Bearer token) + protected
  `GET /api/auth/me` + role gate (USER vs ADMIN). Then Day 8 starts `stockforge-order-service`.
- Device B clones of the repos still unverified (user-owned).

Exact next task:

- Day 7 per `DAY_BY_DAY_GUIDE.md`: `JwtAuthenticationFilter` → protected `/api/auth/me` →
  role gate → tests → curl (login → me with/without token) → push auth → update state here → push.

Commands to run:

- `git pull` in stockforge-project-context and stockforge-auth
- `.\mvnw test` then `.\mvnw spring-boot:run` (JDK 21+; this device has 26)
- curl: login → copy JWT → `curl.exe -H "Authorization: Bearer <token>" http://localhost:8080/api/auth/me`

Files to inspect:

- `stockforge-auth/src/main/java/com/stockforge/auth/` — user/, auth/, dto/, security/

---

## Session 6 — `stockforge-auth`: register/login with bcrypt + JWT (2026-08-08)

Project:        StockForge
Current phase:  Phase 1 — Git/GitHub organization and repository foundation
Current day:    Day 6 (DONE)
Current repo:   stockforge-auth (new) — state repo updated in parallel
Current branch: main
Previous commit: n/a (new repo) — first commit `1e3116d`, bug-fix commit `2306f08` (permit `/error`,
+ `test-auth.ps1`)

What was implemented:

- Repo `stockforge-auth` created (user made empty repo on GitHub) and pushed (`1e3116d`).
- Spring Boot 4.1.0 scaffold via `start.spring.io` (web + actuator + security + validation,
  Java 21 language level, Maven Wrapper). Boot 4 dependency id for the web starter is `web`
  (not `webmvc`) on start.spring.io.
- bcrypt password hashing (`BCryptPasswordEncoder`); users in an in-memory `UserStore`.
- `POST /api/auth/register` → 201 + user (no password field); duplicate email → 409;
  short password (< 8) → 400 (jakarta validation). `POST /api/auth/login` → 200 + signed
  JWT (jjwt 0.13.0, HS256, sub = email, 1h expiry) + user; bad credentials → same 401.
- `JwtService` (sign + parse; tampered tokens rejected), `SecurityConfig` (stateless, no
  CSRF/form login; `/api/auth/*` + health permitted, everything else locked for now).
- Tests: MockMvc (register 201, duplicate 409, login 200 + JWT, wrong password 401, short
  password 400) + JwtService unit tests — all 8 pass. curl-verified live: 201→409→200→401→400.
- README written; `run.log`/`run.err.log` added to `.gitignore` pre-emptively.
- **NEW WORKFLOW RULE baked into `START_OF_DAY.md`:** explain-first — the AI delivers the
  full day briefing and WAITS for acknowledgment before any code/commands (user tests
  manually alongside).

What was learned:

- **Spring Boot 4 validation needs its own starter:** `jakarta.validation` annotations
  (`@NotBlank`, `@Email`, `@Size`, `@Valid`) are NOT bundled with the web starter in Boot 4 —
  add `spring-boot-starter-validation`.
- **Test isolation is a real bug class:** MockMvc tests share one Spring context + one
  `UserStore`, so tests that reused the same email saw leftovers → `expected 201 but was 409`.
  Fix: each test uses its own data. Production suites must never depend on test order.
- **Leftover processes squat ports:** a Day 5 `stockforge-api` JVM was still listening on
  :8080 and the Day 6 server failed to start. Fix: identify via
  `Get-NetTCPConnection -LocalPort 8080` + stop the owning process, and always stop
  `spring-boot:run` servers when done.
- Same-oracle security: returning the same 401 for unknown email and wrong password avoids
  user-probing (account enumeration).

Current problem / open questions:

- None blocking. JWT protects nothing yet — Day 7 adds the filter + protected endpoint.

Incomplete work (record exactly; next session continues from here):

- Day 7: roles + `JwtAuthenticationFilter` (SecurityContext from Bearer token) + protected
  `GET /api/auth/me` + a role gate. Then Day 8 starts `stockforge-order-service`.
- stockforge-auth has no persistence (in-memory users), no logout/refresh/revocation.
- stockforge-web static; stockforge-api has no JWT wiring; Device B clones unverified.

Exact next task:

- Day 7 per the Day 7 prompt in CURRENT_STATE.md: `JwtAuthenticationFilter` → protected
  `/api/auth/me` → role gate (`USER`/`ADMIN`) → tests → curl (login → me with/without token)
  → push → update state here → push.

Commands to run:

- `git pull` in stockforge-project-context (+ any repo being touched)
- `.\mvnw test` then `.\mvnw spring-boot:run` (JDK 21+; this device has 26)
- curl: login → copy JWT → `curl.exe -H "Authorization: Bearer <token>" http://localhost:8080/api/auth/me`

Files to inspect:

- `stockforge-auth/src/main/java/com/stockforge/auth/` — user/, auth/, dto/, security/
- `stockforge-contracts/contracts/openapi.yaml` — auth schemas the service must satisfy
- `project-context/CURRENT_STATE.md` (Day 7 prompt), `project-context/ISSUES_LOG.md`

Expected result:

- `stockforge-auth` has a protected `/api/auth/me`; valid JWT → 200 + user; missing/invalid
  → 401; role gate demonstrable; all tests green.

Long-term direction:

- Keep contract-first; the JWT verification built on Day 7 is what `stockforge-api` reuses
  so every protected endpoint trusts the token. GitHub Actions + Jenkins both first-class
  later (Phase 14-15); containers (Phase 12-13) end cross-device toolchain drift.

Git verification: committed (yes, both repos)  pushed (yes)  verified (yes)

---

## Day 0 — Foundation Session (2026-08-06)

What was implemented:

- `MASTER_PROMPT.md` — the complete StockForge scenario prompt (all 47 rules: learning approach, GitHub org, repo responsibilities, two CI/CD systems, HFT evolution, two-device git workflow, mandatory end-of-day sequence, new-session startup protocol).
- `PROJECT_CONTEXT.md` — permanent architecture/context skeleton (all `PLANNED`).
- `CURRENT_STATE.md` — Day 0 state.
- This file + `adr/` folder.

What was learned:

- Project continuity lives in GitHub, not AI memory.
- First session produces architecture only, and waits for approval.

Exact next task:

- Create GitHub organization `StockForge` + repo `stockforge-project-context`, add remote, push.
- Produce Phase 0 architecture deliverable (Master Prompt §46) into `PROJECT_CONTEXT.md`, then **WAIT FOR APPROVAL**.

Git verification: committed (yes)  pushed (no — no remote yet)  verified (no)

---

## Day 1 — Phase 0 Architecture Proposal (2026-08-06)

What was implemented:

- Wrote the full **Phase 0 architecture proposal** into `PROJECT_CONTEXT.md` (all sections marked `PROPOSED`): target architecture, repo layout (**one service = one folder = one repo**), proposed tech stack, service boundaries, API catalogue + critical traversal, event catalogue, data layer, CI/CD division (GitHub Actions + Jenkins), performance strategy, observability/SLO, security, infrastructure, incident engineering, HFT evolution, phases + repo creation order, 30-day roadmap, risks, ADR index.
- Created `adr/0001-github-org-and-repo-layout.md` and `adr/0002-technology-stack.md`.
- Created `project-context/DAY_BY_DAY_GUIDE.md` — manual GitHub setup guide + folder structure + day-by-day plan with production thinking.

What was learned:

- Architecture must be approved before implementation (Master Prompt §46).
- Backend language is the key approval point (Java 21 vs Go).

Exact next task:

- **Get user approval** of the Phase 0 proposal (especially backend language).
- Set up GitHub org + remote + push (`git remote add origin … ; git push -u origin main`).
- Clone on device B; practice the two-device workflow.
- Then begin Phase 1 repo foundation.

Result: **architecture APPROVED 2026-08-06** (Java 21 + Spring Boot confirmed). Next session = **Day 2** (GitHub org + repo foundation) per `DAY_BY_DAY_GUIDE.md`.

Git verification: committed (yes)  pushed (no — no remote yet)  verified (no)

---

## Day 1.5 — Architecture Approval + Day-by-Day Guide (2026-08-06)

What was implemented:

- Phase 0 architecture **APPROVED** by the user (stack as proposed: Java 21 + Spring Boot, React+TS, PostgreSQL, Redis, Kafka, Docker, kind→EKS, GitHub Actions + Jenkins, k6, Prometheus/Grafana).
- ADR 0001 + 0002 status → ACCEPTED.
- Created `DAY_BY_DAY_GUIDE.md` (manual: GitHub org/repo setup, folder structure, day plan with production thinking).
- Updated `PROJECT_CONTEXT.md` statuses to APPROVED and `CURRENT_STATE.md` to reflect approval.

What was learned:

- The user does GitHub/repo creation manually; the AI builds locally and teaches.

Exact next task (Day 2):

- User creates GitHub org `StockForge` + repo `stockforge-project-context` (empty), runs:
  ```
  git remote add origin https://github.com/Stock-Forge/stockforge-project-context.git
  git push -u origin main
  ```
- Clone on Device B; practice two-device git round-trip.
- Then Phase 1 (stockforge-contracts, Day 3).

Git verification: committed (yes)  pushed (no — no remote yet)  verified (no)

---

## Day 2 — GitHub org + repo foundation + git workflow (2026-08-06)

What was implemented:

- GitHub org **`Stock-Forge`** created (hyphenated; product name stays `StockForge`).
- Repo `stockforge-project-context` created (empty), remote `origin` added, `main` pushed + tracking set (`git push -u origin main`).
- Docs updated to `Stock-Forge` URLs.
- 30-minute roadmap expanded to Day 39+ (PROJECT_CONTEXT.md §21).
- Git workflow practiced: branch → commit → push -u → merge (fast-forward) → push → delete local+remote branch.
- `CHANGELOG.md` created with Day 0-2 entries.
- Commit `89d4b4f` on `origin/main`.

What was learned:

- Push output on PowerShell shows as "git :" red text — that is stderr formatting, not an error; verify with `git status -sb` and `git ls-remote --heads origin`.
- Feature branches merge cleanly when main hasn't moved (fast-forward).

Exact next task (Day 3):

- **User:** create empty repo `stockforge-contracts` in `Stock-Forge`.
- **AI:** create local `stockforge-contracts` (own repo), OpenAPI spec + 7 event schemas + README, push; update state + CHANGELOG here, push.

Git verification: committed (yes)  pushed (yes)  verified (yes)

---

## Day 3 — `stockforge-contracts` contracts (2026-08-06)

What was implemented:

- Repo `stockforge-contracts` created (user made empty repo on GitHub) and pushed (`e1d65cb`).
- `contracts/openapi.yaml` — OpenAPI 3.0.3, 10 paths (auth, market-data, orders, portfolio), 14 schemas, JWT bearer. Validated (pure YAML — first version wrapped in Markdown failed validation).
- `contracts/events/` — 7 event contracts + INDEX (topics, partition keys, idempotency, failure handling).
- README explaining contract-first + contract rules.

What was learned:

- A `.yaml` file must be pure YAML; Markdown fenced blocks are invalid in `.yaml`.
- YAML validation: `python -c "import yaml; yaml.safe_load(open(...))"`.
- Contract-first: interfaces pinned before code; version, don't break.

Exact next task (Day 4):

- **User:** create empty repo `stockforge-web`; Node v24 already installed.
- **AI:** scaffold Vite React+TS app (login placeholder + dashboard), explain structure,
  push, then update state HERE and push.

Git verification: committed (yes, both repos)  pushed (yes)  verified (yes)

---

## Day 4 — `stockforge-web` React trading UI scaffold (2026-08-07)

What was implemented:

- Repo `stockforge-web` created (user made empty repo on GitHub) and pushed (`8e7d075`).
- Vite + React 19 + TypeScript scaffold (`npm create vite@latest . -- --template react-ts`),
  template explained: `package.json` (scripts: dev/build/lint/preview), `tsconfig*`,
  `index.html` (mounts React into `#root`), `src/main.tsx` + `App.tsx`, oxlint config.
- Page shell: top bar with StockForge brand + "API not connected yet" badge; login
  placeholder (disabled form); dashboard with API connection + market data status cards.
- Replaced template demo (counter/hero) styles with a clean dark theme; removed unused assets.
- Verified: `npm run lint` (oxlint) ✅, `npm run build` (tsc -b && vite build) ✅,
  `npm run dev` → HTTP 200 at `http://localhost:5173` with `#root` present (server stopped after).
- Own README written: status, getting started, scripts, structure, API contract, limitations.

What was learned:

- Scaffold truth on this machine: Node v20.19.6 (state previously said v24 — discrepancy
  noted, no impact; v20 is still LTS and works with Vite 8).
- Vite template now ships React 19 + Vite 8 + oxlint (not the older ESLint stack).
- Frontend repo owns its UI/deploy/CI; it consumes the OpenAPI contract in stockforge-contracts.

Current problem / open questions:

- None blocking. Login/dashboard are static; real auth + API calls come later.

Incomplete work (record exactly; next session continues from here):

- stockforge-web is a static shell: no routing, no API wiring, no auth, no tests, no CI.
- Device B clone of stockforge-web not yet verified (user-owned).

Exact next task:

- Day 5: `stockforge-api` — Spring Boot gateway scaffold. User: create empty repo on
  GitHub + install JDK 21 / Maven. AI: `/api/health`, structured logging, correlation-ID
  filter, one MockMvc test; run tests, push, update state here, push.

Commands to run:

- `git pull` in stockforge-project-context (+ any repo being touched)
- `java -version` / `mvn -version` to confirm tooling
- `./mvnw spring-boot:run` then `Invoke-WebRequest http://localhost:8080/api/health`

Files to inspect:

- `stockforge-web/README.md`, `src/App.tsx`, `src/index.css`
- `stockforge-contracts/contracts/openapi.yaml` (the contract the API must satisfy)

Expected result:

- `stockforge-api` on GitHub; `GET /api/health` returns 200 JSON; correlation ID logged.

Long-term direction:

- Keep contract-first: services built against `stockforge-contracts`, UI consumes the
  same spec. GitHub Actions + Jenkins both first-class later (Phase 14-15).

Git verification: committed (yes)  pushed (yes)  verified (yes)

---

## Day 5 — `stockforge-api` Spring Boot gateway scaffold (2026-08-07)

Project:        StockForge
Current phase:  Phase 1 — Git/GitHub organization and repository foundation
Current day:    Day 5 (DONE)
Current repo:   stockforge-api (new) — state repo updated in parallel
Current branch: main
Previous commit: n/a (new repo) — first commit 3374c38, second b0788bb

What was implemented:

- Repo `stockforge-api` created (user made empty repo on GitHub) and pushed
  (`3374c38` scaffold, `b0788bb` gitignore fix). Local repo is its own git repo.
- Spring Boot **4.1.0** scaffold via `start.spring.io` (webmvc + actuator, Java 21
  language level, Maven Wrapper pinned to Maven 3.9.16). No Maven install needed.
- `GET /api/health` → `{"status":"UP"}` (200), verified with `curl` while running.
- `application.yml` with key=value structured logging pattern carrying `[correlationId=...]`.
- `CorrelationIdFilter` (`@Order(1)`): reads `X-Correlation-Id` or generates a UUID,
  stores it in SLF4J MDC, echoes it on the response.
- Tests: MockMvc `/api/health` + 2 filter unit tests — all 4 pass.
- README written; stray `run.log` removed + gitignored (commit `b0788bb`).
- Stack update recorded in ADR 0002 + PROJECT_CONTEXT §5: Spring Boot 3.x → **4.1.0**
  (start.spring.io is 4.x-only now).

What was learned:

- **Spring Boot 4 changes:** `@AutoConfigureMockMvc` moved to package
  `org.springframework.boot.webmvc.test.autoconfigure` (Boot 4 reorganized test modules
  into per-application-type starters like `spring-boot-starter-webmvc-test`).
- **start.spring.io version labels vs Maven Central:** the generator reported
  `4.1.0.RELEASE`, but Maven Central has `4.1.0` — the parent POM failed to resolve
  until we dropped the `.RELEASE` suffix. Always verify against
  `repo.maven.apache.org` metadata.
- The Maven Wrapper makes JDK/Maven setup portable: JDK 26 here satisfies the JDK 21+
  requirement; the compiler targets language level 21.

Current problem / open questions:

- None blocking. This machine now has JDK 26 (state previously said Java 17 — resolved).
- Personal PC toolchain (JDK 21 vs 26) not verified; both satisfy java.version=21.

Incomplete work (record exactly; next session continues from here):

- stockforge-api talks to nothing yet (no auth, no persistence, no routing).
- stockforge-web still a static shell — no API wiring.
- Device B clone of the repos not yet verified (user-owned).
- Contract tooling + contract tests deferred (service phases / Phase 14).

Exact next task:

- Day 6: `stockforge-auth` — register + login with bcrypt + JWT. User: create empty
  repo on GitHub. AI: Spring Boot service reusing the Day 5 pattern (health, correlation
  ID), in-memory users, `POST /api/auth/register` + `/login`, wrong password → 401,
  JWT on success, tests, push, update state here, push.

Commands to run:

- `git pull` in stockforge-project-context (+ any repo being touched)
- `.\mvnw test` then `.\mvnw spring-boot:run` (JDK 21+; this device has 26)
- `curl http://localhost:8080/api/health` (Day 5 app) / auth endpoints (Day 6)

Files to inspect:

- `stockforge-api/` — pom.xml, HealthController, CorrelationIdFilter, application.yml
- `stockforge-contracts/contracts/openapi.yaml` — auth paths (`/api/auth/register`,
  `/api/auth/login`) the Day 6 service must satisfy
- `project-context/CURRENT_STATE.md` (Day 6 prompt), `project-context/ISSUES_LOG.md`

Expected result:

- `stockforge-auth` on GitHub; register → login returns a JWT; wrong password = 401.

Long-term direction:

- Keep contract-first: services built against `stockforge-contracts`, UI consumes the
  same spec. GitHub Actions + Jenkins both first-class later (Phase 14-15); containers
  (Phase 12-13) end cross-device toolchain drift.

Git verification: committed (yes, both repos)  pushed (yes)  verified (yes)

---

## Day 5 partial — tooling gate + prompt/features upgrade (2026-08-07)

Project:        StockForge
Current phase:  Phase 1 — Git/GitHub organization and repository foundation
Current day:    Day 5 (started, BLOCKED at tooling gate — deferred to personal PC)
Current repo:   stockforge-project-context (state); Day 5 build moves to the personal PC
Current branch: main
Previous commit:19166d2 (prompt/features) → new closeout commit

What was implemented:

- Prompt upgrades (pushed 19166d2): every session must begin with a FULL-detail briefing —
  what we're doing today + how it helps, every step with exact commands, the tech stack
  used at each step, and a comparison to production-grade trading platforms (Zerodha/Groww).
- New `project-context/ISSUES_LOG.md`: per-day log of every issue (symptom → cause → fix →
  production relevance), auto-updated at the end of every session. Seeded with Day 3 & Day 4.
- Auto-update rule made explicit across START_OF_DAY.md, SESSION_PROMPTS.md, the guide,
  CURRENT_STATE.md, and README: state files update automatically at end of session.
- Day 5 startup protocol run: this device has Java 17.0.12 + no Maven + no full admin →
  JDK 21 install is not viable here. Decision: do Day 5 on the personal PC.

What was learned:

- "Works on my machine" is a real problem: toolchain (JDK 21), admin rights, and Maven
  differ per device. Fixes: Maven Wrapper pins Maven; JDK 21 setup happens on the
  admin machine; containers (Phase 12-13) make toolchains fully device-independent.
- A PowerShell command that included `curl.exe ... https://api.adoptium.net/...` was
  blocked by AMSI/antivirus as "malicious content" (false positive). Lesson: run security
  tooling-sensitive steps (downloads) on the admin machine, or split into smaller steps.

Current problem / open questions:

- None blocking. Day 5 waits for JDK 21 on the personal PC.

Incomplete work (record exactly; next session continues from here):

- Day 5 `stockforge-api` scaffold NOT built — needs JDK 21 on the personal PC.
- stockforge-web still static (no API wiring); its repo is pushed (`8e7d075`).
- Device B (personal PC) clone of the repos not yet verified.

Exact next task:

- On the personal PC (full admin): clone project-context + web; install Temurin JDK 21;
  create empty repo `stockforge-api`; run Day 5 per the Day 5 prompt in CURRENT_STATE.md
  (`/api/health`, structured logging, correlation-ID filter, one MockMvc test), push,
  then update state here and push.

Commands to run:

- `git clone https://github.com/Stock-Forge/stockforge-project-context.git`
- `git clone https://github.com/Stock-Forge/stockforge-web.git`
- `java -version` (expect 21)
- `.\mvnw spring-boot:run` then `Invoke-WebRequest http://localhost:8080/api/health`

Files to inspect:

- `project-context/CURRENT_STATE.md` (Day 5 prompt), `project-context/ISSUES_LOG.md`
- `stockforge-contracts/contracts/openapi.yaml` (the contract the API must satisfy)

Expected result:

- `stockforge-api` on GitHub; `GET /api/health` returns 200 JSON; correlation ID logged.

Long-term direction:

- Keep contract-first; GitHub Actions + Jenkins both first-class later (Phase 14-15);
  containers (Phase 12-13) end cross-device toolchain drift.

Git verification: committed (yes)  pushed (yes)  verified (yes)

---

## Day 3 — Upcoming: `stockforge-contracts` (next session pointer)

Current state at the end of Day 2 / before Day 3 (see `CURRENT_STATE.md`):

- Repo `stockforge-project-context` on `origin/main` at `cb9af31`, clean tree, 9 commits.
- Central-state rule enforced: all state updates happen in this repo, never in the
  working repo; both repos are committed + pushed each day.
- Next: Day 3 — create repo `stockforge-contracts` (user creates empty repo on GitHub),
  AI builds `contracts/openapi.yaml` + `contracts/events/` (7 events) + README in a new
  local repo, pushes it, then updates THIS repo's state to point at Day 4.
- Begin with the full `START_OF_DAY.md` paste.

Git verification: not applicable (pointer entry — no code changed).
