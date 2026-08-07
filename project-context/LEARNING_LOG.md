# STOCKFORGE — DAY-WISE LEARNING LOG

> One entry per project day, written at the END of every session (mandatory, per
> `START_OF_DAY.md` Phase 3). Purpose: a plain-language record of **what each day
> taught**, so you can re-read any day later and remember the concept without hunting
> through code. This file is your personal study notes — not architecture, not status
> (those live in `PROJECT_CONTEXT.md` / `CURRENT_STATE.md`).
>
> **How to use:** skim the day you want. Each entry has the same shape:
> 1. What we built 2. The concept in one sentence 3. What I should remember 4. Production/HFT relevance.

---

## Entry template (for future days — copy this)

```
### Day N — <topic>

- **What we built:** ...
- **Concept in one sentence:** ...
- **What I should remember:** ...
- **Production/HFT relevance:** ...
- **My own words (teach-back):** ...
```

---

## Day 0 — Project foundation (2026-08-06)

- **What we built:** the master prompt, project context, current-state file, session
  prompts, and the ADR folder. No application code — just the machinery that lets a
  brand-new AI session (or you, next week) reconstruct the whole project from files.
- **Concept in one sentence:** an engineering project is only as good as its
  documentation — the project's memory lives in versioned files, not in anyone's head.
- **What I should remember:** every session starts by reading files (`git pull` →
  read context/state → reconcile), never by assuming.
- **Production/HFT relevance:** real teams keep runbooks, architecture docs, and ADRs
  for the same reason — a 10-person HFT team must onboard, hand over, and audit decisions.

## Day 1 / 1.5 — Architecture proposal + approval (2026-08-06)

- **What we built:** the full Phase 0 architecture (9 services, repo layout, stack,
  roadmap, ADRs 0001 + 0002) and got it **approved** before any code.
- **Concept in one sentence:** architecture is a decision record — write down *why* you
  chose each technology so future engineers don't re-litigate it.
- **What I should remember:** the stack (Java 21 + Spring Boot, React+TS, PostgreSQL,
  Redis, Kafka, Docker→K8s, GitHub Actions + Jenkins, k6, Prometheus/Grafana) and the
  repo rule (one service = one folder = one repo).
- **Production/HFT relevance:** before a platform exists on paper, nothing should be
  built. ADRs are how real firms stay consistent while teams churn.

## Day 2 — GitHub org + git workflow (2026-08-06)

- **What we built:** the `Stock-Forge` org, pushed `stockforge-project-context`, and
  practiced the branch → commit → push → merge round-trip.
- **Concept in one sentence:** git is the single source of truth — everything is
  recoverable from GitHub, on any device.
- **What I should remember:** `git push -u origin main` sets tracking; PowerShell shows
  git's stderr as red "git :" text that is NOT an error — verify with `git status -sb`.
- **Production/HFT relevance:** two-device git work is the miniature of distributed
  teams; HFT firms run the same pull→code→push loop, just with PRs + branch protection.

## Day 3 — `stockforge-contracts` (2026-08-06)

- **What we built:** `openapi.yaml` (10 paths, 14 schemas, JWT bearer) + 7 Kafka event
  contracts (`OrderCreated`, `OrderAccepted`, `OrderRejected`, `OrderExecuted`,
  `OrderCancelled`, `PositionUpdated`, `MarketPriceUpdated`) + README.
- **Concept in one sentence:** contract-first — pin the *interfaces* between services
  before writing any service, so changes can't silently break each other.
- **What I should remember:** a `.yaml` file must be pure YAML (no Markdown fence);
  validate with `python -c "import yaml; yaml.safe_load(open(...))"`.
- **Production/HFT relevance:** real platforms version their API catalogs and event
  schemas (Confluent Schema Registry) and gate on contract tests in CI; the order
  lifecycle is the heart of trading, and its events are the shared language.

## Day 4 — `stockforge-web` (2026-08-07)

- **What we built:** a Vite + React 19 + TypeScript scaffold with a login placeholder
  and dashboard status cards, dark StockForge theme.
- **Concept in one sentence:** the frontend is its own repo with its own life (dev
  server, build, lint) that will consume the API contracts later.
- **What I should remember:** `npm create vite@latest . -- --template react-ts`;
  `index.html` mounts React into `#root`; `src/main.tsx` + `App.tsx` are the entry;
  verified with `npm run lint`, `npm run build`, `npm run dev` (HTTP 200 at :5173).
- **Production/HFT relevance:** a trading UI is the human window onto the platform —
  but in HFT the UI is a *consumer*; speed lives in the backend. Know which is which.

## Day 5 — `stockforge-api` (2026-08-07)

- **What we built:** the first backend repo — Spring Boot 4.1.0 with `GET /api/health`
  (→ `{"status":"UP"}`), structured logging (`[correlationId=...]` in every line), and
  a correlation-ID filter (MDC + `X-Correlation-Id` echo), 4 passing tests.
- **Concept in one sentence:** a gateway is the single door into the platform, and it
  must be observable from day one — health for machines, correlation IDs for humans,
  structured logs for tools.
- **What I should remember:**
  - Spring Boot = one `@SpringBootApplication` class; controllers map URLs to methods;
    filters run before/after every request (`@Order(1)` + `OncePerRequestFilter`).
  - Maven Wrapper (`mvnw`) pins Maven — no global install needed.
  - Stack update: Spring Boot went 3.x → **4.1.0** (start.spring.io is 4.x-only);
    Boot 4 moved `@AutoConfigureMockMvc` to `org.springframework.boot.webmvc.test.autoconfigure`.
  - start.spring.io labels versions `4.1.0.RELEASE` but Maven Central publishes `4.1.0` —
    verify against `maven-metadata.xml` if the parent POM won't resolve.
- **Production/HFT relevance:** health endpoints drive auto-scaling/self-healing (a
  dead instance is replaced without humans); correlation IDs let you trace one request
  across every service in microseconds — the ancestor of the tracing you'll use to
  prove HFT latency wins later. Logs are queried by machines, not read by eye.

## Day 5 — Bonus deep-dive: how health checks work in production & HFT

- **What we explored:** a Q&A follow-up on Day 5 — how `/actuator/health` checks
  dependencies (DB/Kafka/Redis), and how that pattern changes in true HFT.
- **Concept in one sentence:** health = *"know your status instantly and act
  automatically"*; web apps poll+restart, HFT uses passive heartbeats + safety-first halt.
- **What I should remember:**
  - **Dependency probes:** DB → `SELECT 1`; Redis → `PING`; Kafka → broker metadata
    (cluster). One DOWN component makes the whole status DOWN → instance pulled from rotation.
  - **The zombie problem:** a process can be "UP" while useless (e.g. stale market data).
    Health checks exist to catch exactly this — dependency-aware, not just process-alive.
  - **Cloud/web pattern:** poll + probe on demand → restart / pull from load balancer.
    Tolerance in seconds, retries OK. This is what StockForge builds Days 0-25.
  - **True HFT pattern (Phase 26):** passive heartbeats on a separate management plane
    (no HTTP polls on the hot path); **latency is itself the health signal**; market-data
    **sequence-gap detection** catches stale feeds in microseconds; action = **kill switch /
    go flat** (cancel orders, stop quoting) rather than restart; **active/standby warm
    twins** for failover instead of slow resync. Never trade on stale state.
- **Production/HFT relevance:** the web model is "detect → repair"; HFT is
  "detect → halt + failover", because a restart takes seconds you don't have. Both are
  the same idea, implemented for different latency budgets. We need the k6 baselines and
  observability built in Days 0-25 before the HFT differences can even be measured.

---

## How the next day continues

The current day and its exact plan are always in `CURRENT_STATE.md` (see the
"NEXT DAY PROMPT" section). This log only records what each completed day taught.
