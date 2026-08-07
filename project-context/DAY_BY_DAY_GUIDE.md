# STOCKFORGE — DAY-BY-DAY GUIDE

> Your manual companion. Read the section for the *current day* before starting.
> You do the GitHub/org steps by hand; the AI session does the building on your machine.
> This file is version-controlled — it syncs across both devices via `git pull`.
>
> Each day = roughly **30 minutes**. Each day explains: **what we build, why**
> (including production-level thinking), the **steps**, the **expected result**,
> and the **production lesson**.

---

## HOW TO USE THIS GUIDE

1. Before starting an AI session: `git pull` (on the device you're using).
2. **Copy-paste the `START_OF_DAY.md` prompt into the new AI session** — it makes the AI
   reconstruct the project from files, work today's 30-minute day, and do the end-of-day
   sequence automatically.
3. Read the **current day** section below (it tells you what the AI session will do and what *you* must do manually).
4. Do your manual steps (e.g. create a GitHub repo, run a command).
5. Let the AI build on your machine, teaching as it goes. It starts by telling you
   **what we're doing today and how it helps the project** — that's your cue to listen,
   ask questions, then approve before it codes.
6. End of day: the AI updates state files, commits, pushes. **You verify the push.**
7. On the other device next time: `git pull` → paste `START_OF_DAY.md` → continue.

The guide grows day by day. Later days are outlines; near-term days are detailed.

---

## PHASE A — ONE-TIME GITHUB SETUP (do this once, on Device A)

This makes GitHub the shared source of truth for both devices.

### A.1 Create your GitHub account (if you don't have one)

1. Go to `https://github.com` → **Sign up** → follow the prompts.
2. Verify your email address (GitHub emails you a confirmation link).

### A.2 Create the GitHub Organization `StockForge`

1. Go to `https://github.com/account/organizations/new`
   (or: avatar → **Settings** → **Organizations** → **New organization**).
2. Plan screen → choose **Free** (Individual) → **Continue**.
3. Fill in:
   - **Organization account name:** `StockForge`
   - If it says the name is taken, use the agreed fallback `StockForge-Trading` and tell the AI so docs can be updated.
   - **Contact email:** your email.
   - "This organization belongs to": *My personal account*.
4. CAPTCHA → **Create organization**.
5. Skip inviting members for now.
6. Remember the org URL: `https://github.com/Stock-Forge`.
   > **Actual org created 2026-08-06:** `Stock-Forge` (hyphenated). The project/product name stays **StockForge**; only the GitHub org is hyphenated. All URLs below use `Stock-Forge`.

> **Production thinking:** in a real company this org would be owned by the engineering org,
> with access controlled by teams (IAM/SAML/SCIM). We keep it under your personal account
> because it's a learning project. The *structure* (org → repos) is exactly how enterprises
> separate business units and components.

### A.3 Create the repo `stockforge-project-context`

1. Go to `https://github.com/Stock-Forge` → **New** (or top-right **+** → **New repository**).
2. Verify the **owner** dropdown says **Stock-Forge** (not your personal username).
3. Fill in:
   - **Repository name:** `stockforge-project-context`
   - **Description:** `StockForge project continuity — state, context, and AI session prompts.`
   - **Visibility:** Public (simpler for two devices) — private is fine too, you'll just sign in during clone.
   - **Leave these UNCHECKED:** "Add a README", ".gitignore", "license" — you already have files locally. Checking them causes a conflict.
4. **Create repository.** Leave the result page open.

> **Production thinking:** repositories in enterprises are *thin slices* of one org,
> each owning one responsibility, each with its own CI and permissions. This repo is
> the "meta" repo — the place the team's shared knowledge (runbooks, ADRs, state) lives.

### A.4 Push the local repo to GitHub (Device A)

In PowerShell, inside the repo folder
(`C:\CODE\HFT Application\stockforge-project-context`):

```powershell
git remote add origin https://github.com/Stock-Forge/stockforge-project-context.git
git push -u origin main
```

**First-time sign-in:** a browser window will open to approve GitHub access.
If instead Git asks for a password, you need a **Personal Access Token**:

1. `https://github.com/settings/tokens` → **Generate new token** → **Generate new token (classic)**
2. Name `git-push`, expiry 90 days, scope: check **repo** → **Generate token**
3. Copy the token (shown only once) and paste it as the password.

**Verify:**

```powershell
git status        # should say clean
git remote -v     # should show your origin URL
git log --oneline # should show Day 1 commit
```

Then refresh the GitHub repo page — you should see `README.md` and `project-context/`.

> **Production thinking:** `git push -u origin main` sets the *upstream*. From then on
> `git push` alone works. Enterprises use **branch protection** on `main` (no direct
> pushes, all changes via PR + required checks) — we'll add that in Phase 14 when
> GitHub Actions exists. For now, single-branch push is fine to learn the mechanics.

### A.5 Set up Device B

1. Install Git: `https://git-scm.com/downloads` (next-next-next).
2. Open PowerShell:

```powershell
cd C:\CODE
git clone https://github.com/Stock-Forge/stockforge-project-context.git
```

3. Verify:

```powershell
cd C:\CODE\stockforge-project-context
git pull
git log --oneline
```

> Choose a consistent local root on BOTH devices (e.g. `C:\CODE`). The repo folders
> must live in the same place relative to the root on both machines, or paths in
> state files get confusing.

### A.6 Daily workflow (both devices, forever)

- **Before work:** `git pull`, then check `git status` / `git branch` / `git log -5`.
- **During:** the AI builds; you learn and approve.
- **End of day:** AI updates `CURRENT_STATE.md` + `SESSION_PROMPTS.md` → commit → push.
  You run `git status` and confirm it's clean and pushed.

---

## PHASE B — LOCAL FOLDER STRUCTURE (ONE SERVICE = ONE FOLDER = ONE REPO)

Every StockForge component is an **independent folder with its own `.git`**, mapped 1:1
to a GitHub repo in the `StockForge` org. **Never nest one repo inside another.**

We create folders **only when the phase needs them** — not all at once.

```
C:\CODE\HFT Application\            (or agreed root; same on both devices)
│
├── stockforge-project-context\      ✅ exists        project continuity
├── stockforge-contracts\            ✅ Day 3          API/event contracts
├── stockforge-web\                  ✅ Day 4          React trading UI
├── stockforge-api\                  Day 5            API gateway
├── stockforge-auth\                 Day 6-7          auth
├── stockforge-order-service\        Day 8-9          orders
├── stockforge-risk-service\         Phase 5          risk limits
├── stockforge-matching-engine\      Phase 6          matching (perf-critical)
├── stockforge-market-data\          Phase 7          simulated market data
├── stockforge-portfolio\            Phase 8          positions/P&L
├── stockforge-notification\         Phase 8          WS notifications
├── stockforge-performance\          Phase 16         load/perf/baselines
├── stockforge-shared-actions\       Phase 14         reusable CI workflows
├── stockforge-deployment\           Phase 18         K8s manifests/versions
└── stockforge-infrastructure\       Phase 19/23      IaC / cloud
```

When a day says "create the `stockforge-xyz` repo": you create it on GitHub (A.3 pattern,
empty, no README), the AI does `git init` in a new local folder, and we push.

---

## PHASE C — THE DAY PLAN (what we build & why, production thinking)

Legend for "Production": what real companies do for this exact thing.

---

### Day 0 ✅ — Project foundation (DONE)

- Built the master prompt + state/context files.
- **Production:** every serious team has an internal "runbooks / architecture / onboarding"
  repo. That's what `stockforge-project-context` is.

### Day 1 ✅ — Phase 0 architecture (DONE, APPROVED)

- Architecture, stack (Java 21 + Spring Boot, React+TS, PostgreSQL, Redis, Kafka,
  Docker, kind→EKS, GitHub Actions + Jenkins, k6, Prometheus/Grafana).
- **Production:** architecture is reviewed and *approved* before code. ADRs record
  *why* decisions were made so future engineers (and future AI sessions) don't re-litigate.

---

### Day 2 ✅ — GitHub org + repo foundation + Git practice (DONE 2026-08-06)

**Goal:** GitHub org created, repo pushed, second device cloned, workflow proven.

**What & why:** this is the "source of truth" foundation. Everything from now on is
recoverable from GitHub.

**You do (manual):** all of Phase A if not already done (org, repo, push, clone on B).

- ✅ Org `Stock-Forge` created (hyphenated; product name stays `StockForge`).
- ✅ Repo `stockforge-project-context` created, `main` pushed + tracked.
- ✅ Git round-trip practiced on Device A (branch → commit → push → merge → delete branch).
- ⏳ **Still to do — Device B clone:**
  ```
  cd C:\CODE
  git clone https://github.com/Stock-Forge/stockforge-project-context.git
  cd stockforge-project-context
  git log --oneline   # expect: 89d4b4f ... b9e8cb3
  ```

**AI session did:** startup protocol; updated docs to `Stock-Forge`; expanded the
30-minute roadmap to Day 39+; practiced the branch workflow; created `CHANGELOG.md`;
updated state; committed (`89d4b4f`); pushed; verified.

**Expected result:** `git log` on both devices shows the same commits; a new branch
round-trip works.

**Production lesson:** Git is the single source of truth; the workflow you practiced
here (pull → work → commit → push) is the same workflow engineers use daily. The only
difference later is branch protection + PRs instead of direct pushes.

---

### Day 3 ✅ — `stockforge-contracts`: API & event contracts (DONE 2026-08-06)

**Goal:** create the contracts repo with OpenAPI spec + Kafka event schema stubs.

**Result:** repo `stockforge-contracts` created + pushed (`e1d65cb`) with
`contracts/openapi.yaml` (10 paths, 14 schemas) and `contracts/events/` (7 events +
INDEX). Contract-first rule documented in README.

**What we build:** `stockforge-contracts` — the *agreement* between services:
what endpoints exist, what requests/responses look like, what Kafka events carry.
No app code yet, just the contract files + README.

**Why:** 10 services can silently break each other unless the *interface* is pinned
and versioned. Enterprises build **contract-first**: the API spec is written before
the service, so frontend and backend can be built in parallel against the same contract.

**Production:** companies maintain API catalogs (OpenAPI), event registries (AsyncAPI,
Confluent Schema Registry) and enforce **contract tests** in CI so a producer change
can't silently break consumers. We'll build toward that.

**You do:** create repo `stockforge-contracts` on GitHub (empty). AI creates the folder,
the spec, commits, pushes.

**Expected result:** `contracts/` with `openapi.yaml` (auth + orders + portfolio)
and `events/` with the 7 event definitions, plus a README explaining the layout.

**Production lesson:** interfaces are contracts; change them with versioning, never silently.

**Done note:** the `openapi.yaml` lesson — a `.yaml` file must be pure YAML, not Markdown
wrapped in a ```yaml fence (the first version failed validation; rewritten as pure YAML).

---

### Day 4 ✅ — `stockforge-web`: React trading UI scaffold (DONE 2026-08-07)

**Goal:** a working (ugly) web app: Vite + React + TypeScript, page shell with login
placeholder and a dashboard that shows "connected" state.

**What & why:** the browser is where users interact. We scaffold *frontend* first so
there's always something visible; the API comes next day.

**Production:** real trading UIs are React/Vue SPAs talking to a backend over REST +
WebSocket, behind a CDN, with feature flags and telemetry. We start tiny and grow.

**You do:** create repo `stockforge-web` on GitHub; install Node.js (`https://nodejs.org`,
LTS) if not present. AI scaffolds the app with `npm create vite@latest`, explains each
file, commits, pushes.

**Result (done):** repo `stockforge-web` created + pushed (`8e7d075`). Vite + React 19 +
TS scaffold; page shell with login placeholder + dashboard status cards ("API not
connected yet"); dark StockForge theme. `npm run lint` + `npm run build` pass;
`npm run dev` served HTTP 200 at `localhost:5173` (verified, then stopped).
Node on this machine is v20.19.6 (LTS).

**Expected result:** `npm run dev` shows a local site at `localhost:5173` with our shell.

**Production lesson:** a frontend repo owns its UI, its own CI, its own deploy — even
though it talks to many backends.

---

### Day 5 — `stockforge-api`: Spring Boot gateway scaffold

**Goal:** a Spring Boot app with `/api/health`, structured logging, a correlation ID
filter, and a placeholder that talks to nothing yet.

**What & why:** the API is the single client-facing door. Everything in production goes
through it: auth, validation, rate limiting, tracing. We learn Spring Boot's request
lifecycle here — the concepts (filters, beans, config, tests) repeat in every later service.

**You do:** install JDK 21 (`https://adoptium.net`, Temurin 21) and Maven
(`https://maven.apache.org`); create repo `stockforge-api` on GitHub.
AI creates the project, explains `pom.xml`, `@RestController`, filters, `application.yml`,
a unit test; commits, pushes.

**Expected result:** `./mvnw spring-boot:run` serves `http://localhost:8080/api/health`.

**Production lesson:** every production API has health endpoints, structured logs and
correlation IDs from day one — you can't add observability after the fact.

---

### Day 6-7 — `stockforge-auth`: registration & login

**Goal (Day 6):** register + login with bcrypt password hashing and a JWT.
**Goal (Day 7):** roles + an `@Authenticated` check reused by the API.

**What & why:** security foundation. Passwords are never stored in plain text; we issue
signed tokens the API verifies without a DB lookup per request.

**Production:** enterprises use an identity provider (Auth0, Keycloak, Azure AD);
JWTs are the standard token format; rotation, expiry, and revocation are first-class.
We build the *concepts* by hand so you understand what the managed services do.

**You do:** create repo `stockforge-auth`; AI builds service, tests, README, CI-ready.

**Expected result:** `POST /api/auth/register` + `/login` return a JWT; wrong password = 401;
tests pass.

**Production lesson:** identity is the highest-risk area — hashing, token expiry and
least-privilege scopes are non-negotiable.

---

### Day 8-9 — `stockforge-order-service`: orders (in-memory first)

**Goal (Day 8):** create/list/cancel orders with a state machine
(NEW → ACCEPTED → FILLED / CANCELLED), stored in memory.
**Goal (Day 9):** add a mocked risk check before accepting.

**What & why:** the core domain. Orders have a lifecycle; state must be explicit and
testable. In-memory first keeps the 30-minute unit small; the DB comes at Day 10.

**Production:** order lifecycle, idempotency (client order IDs), and event publishing
are the heart of trading systems. Real order services persist every state change and
emit events (Kafka) for downstream systems.

**You do:** create repo `stockforge-order-service`; AI builds, tests, README.

**Expected result:** REST create/list/cancel works; unit tests pass; state transitions
logged.

**Production lesson:** correctness of state transitions and idempotency beat clever code —
this is where trading firms get fined for bugs.

---

### Day 10 — PostgreSQL: schema, migrations, the missing-index lesson

**Goal:** persist orders in PostgreSQL via Flyway migrations; run the *same* query with
and without an index and measure the difference.

**What & why:** databases store truth. We learn schema design, migrations (versioned
schema changes), and why indexes matter — measured, not guessed.

**Production:** every production DB change is a versioned migration; schema changes go
through review; indexes are designed from query patterns; connection pooling and
transaction isolation are tuned. We will deliberately break these later (Phase 9/24)
to learn detection.

**You do:** install PostgreSQL 16 (`https://www.postgresql.org/download/`) — or we use
Docker in Phase 12; create repo for the DB work inside `stockforge-order-service`.
AI writes migration + queries + explains `EXPLAIN ANALYZE`.

**Expected result:** orders survive restart; `EXPLAIN ANALYZE` shows seq-scan vs index-scan.

**Production lesson:** measure before optimizing — a "missing index" is found by
profiling queries, not guessing.

---

## Days 11+ (phases ahead — outline, detail added when reached)

| Days/Phase | Topic | What we build & why (production view) |
|---|---|---|
| 11-15 (Ph 5-8) | risk-service, matching-engine, market-data, portfolio, notification | Real services: risk limits (why firms cap exposure), in-memory order book + price-time matching (perf-critical, the future HFT star), simulated prices via WebSocket (market-data feeds), P&L/positions (bookkeeping that must reconcile), WS notifications (async pushes) |
| 16-18 (Ph 9-10) | DB optimization, Redis | Connection pooling/locks/deadlocks measured; Redis caching + rate limiting vs Postgres-only, measured |
| 19-21 (Ph 11) | Kafka events | OrderCreated/Accepted/Rejected/Executed/Cancelled, PositionUpdated, MarketPriceUpdated; consumer groups, lag, idempotency, DLQ; we intentionally break it |
| 22-24 (Ph 12-13) | Docker + Compose | Every service containerized; compose runs the whole platform locally; each Dockerfile explained |
| 25-28 (Ph 14) | GitHub Actions | PR checks: lint → unit → integration → contract → security → build → image; reusable workflows in shared-actions; branch protection |
| 29-33 (Ph 15) | Jenkins | Controller/agents, Jenkinsfile pipeline, credentials, shared library, scheduled/parameterized jobs; the *equivalent* pipeline to GitHub Actions so you see why enterprises run both |
| 34-38 (Ph 16-17) | Performance testing + gates | k6 load tests, baselines, thresholds, perf regression gates in CI; the skill you're bringing to work |
| 39-42 (Ph 18-19) | Kubernetes + deployment automation | kind cluster: Pod/Deployment/Service/Probes/ConfigMap/Secret/Ingress/HPA; deployment repo pins service versions per env; rolling deploys |
| 43-46 (Ph 20-21) | Observability + SLO/SRE | Prometheus/Grafana/OTel/Loki; SLI/SLO/error budgets from real measured baselines |
| 47-49 (Ph 22) | Security hardening | Secrets, HTTPS, scanning, least privilege, audit logs |
| 50-53 (Ph 23-25) | Cloud, failure injection, rollout | AWS/EKS/IaC; deliberate incidents (DB slowdown, Redis failure, Kafka lag, CPU, memory leak, bad deploy); rollback, canary/blue-green |
| 54+ (Ph 26) | HFT evolution | Order-book optimization, lock contention, GC, object pooling, CPU affinity, lock-free structures, binary protocols, ns/µs measurement — every change baseline→hypothesis→change→measure→conclude |

> Day numbers are approximate — we add a new day when the previous one is done. The
> roadmap table in `PROJECT_CONTEXT.md` §18 and §21 is the master list.

---

## PRODUCTION MINDSET CHEAT-SHEET (read anytime)

For every feature, think in three columns:

| Local (we build) | Production (enterprises do) | Difference (what changes) |
|---|---|---|
| Local PostgreSQL | AWS RDS | Managed, HA, backups, IAM |
| Docker / Compose | Kubernetes | Orchestration, auto-scaling, self-healing |
| Local Kafka (Docker) | Managed Kafka (Confluent/MSK) | Capacity, replication, schema registry |
| Local perf test (k6) | Distributed load generation | Scale of load, network realism |
| Direct push to main | Branch protection + PR + required checks | Review + gates before merge |
| Manual deploy | CI/CD pipeline + canary/rollback | Repeatable, reversible, approved |
| App logs | Prometheus/Grafana + OTel + SLOs | Alerting on targets, error budgets |
| Single machine | Multi-AZ/region, observability | Failure domains, capacity planning |

Rule of thumb: **we first build it locally and understand it, then we ask
"what would production add?"** — never the other way around.

---

## HOUSEKEEPING

- **Central-state rule:** ALL project state lives in `stockforge-project-context`.
  When we work in a different repo (e.g. `stockforge-contracts`), that repo receives
  only its own code/tests/README — state updates (`CURRENT_STATE.md`,
  `SESSION_PROMPTS.md`, `PROJECT_CONTEXT.md`, `DAY_BY_DAY_GUIDE.md`, `ISSUES_LOG.md`,
  `CHANGELOG.md`) happen HERE and are pushed HERE. Both repos are committed and pushed each day.
- **Auto-update rule:** at the end of EVERY session the AI automatically updates all of
  the files above (state, prompts, guide, issues log, changelog) — it does not wait to
  be asked. If the AI finishes without updating them, the session is NOT complete.
- **Commit messages describe the change, not the day.** Write what was actually done
  (e.g. "Add OpenAPI spec for orders API"), not "Day 3". Day numbers only appear in
  `CURRENT_STATE.md` / `SESSION_PROMPTS.md` / `CHANGELOG.md`.
- **Never push without the day's state files updated** (CURRENT_STATE.md,
  SESSION_PROMPTS.md). The AI does this; you verify.
- **If you see unexpected changes after `git pull`: STOP** and ask the AI to
  investigate. Never force-push.
- **If the org name `StockForge` is taken**, use `StockForge-Trading` and tell the AI
  to update this guide + PROJECT_CONTEXT.md.
- Each day is 30 minutes. **Stop at the clean point even if it feels small.**
