# PROJECT_CONTEXT.md

> Permanent architecture/context document for **StockForge**.
> A completely new AI must understand the whole project from this file.
> Keep this file up to date. Never let it become stale.
>
> Status legend: `PLANNED` | `PROPOSED` | `APPROVED` | `IMPLEMENTED` | `IN PROGRESS` | `DEPRECATED`
> `PROPOSED` = written in Phase 0. `APPROVED` = accepted by user on 2026-08-06 (Day 1).

---

## 1. Project Purpose

Build **StockForge** — a production-style stock trading platform and engineering laboratory. Long-term learning and engineering program for someone who is a Performance Engineer but relatively new to software development, backend/frontend, Git/GitHub, CI/CD, Jenkins, Docker, Kubernetes, cloud, IaC, and distributed systems.

The project deliberately teaches **WHY** each technology exists, not just its commands.

**Status:** `IMPLEMENTED` (context documented) / project `APPROVED`

---

## 2. Final Goal

A complete production-grade stock trading platform, understood end-to-end (request/event/service/infrastructure lifecycle), automated delivery and performance validation through **both GitHub Actions and Jenkins**, deployed and operated on Kubernetes, with SLOs and observability, deliberate production-failure injection/resolution — then progressive evolution toward **high-throughput / low-latency HFT-style architecture**.

**Status:** `APPROVED` (2026-08-06)

---

## 3. Target Architecture

### 3.1 System overview (Phase 2-25 end state)

```
                    ┌─────────────────────────────┐
                    │   Browser  (stockforge-web) │
                    └──────────────┬──────────────┘
                                   │ REST + WebSocket
                                   ▼
                    ┌─────────────────────────────┐
                    │  stockforge-api  (gateway)  │  auth integration, validation,
                    └──────────────┬──────────────┘  rate limiting, correlation ID
                                   │
              ┌────────────────────┼───────────────────────┐
              ▼                    ▼                       ▼
   ┌────────────────┐   ┌────────────────┐      ┌────────────────────┐
   │ stockforge-auth │   │ order-service  │      │ market-data (WS)   │
   │  users/tokens   │   │  orders/state  │      │ price simulation    │
   └────────────────┘   └───────┬────────┘      └──────────┬─────────┘
              ┌─────────────────┼──────────────────────────┤
              ▼                 ▼                          ▼
   ┌────────────────┐   ┌────────────────┐      ┌────────────────────┐
   │ risk-service   │◄──│ order-service  │      │ matching-engine     │
   │ limits/checks  │   └───────┬────────┘      │ order book + match  │
   └────────────────┘           │               └──────────┬─────────┘
              ┌─────────────────┼──────────────────────────┤
              ▼                 ▼                          ▼
   ┌────────────────┐   ┌────────────────┐      ┌────────────────────┐
   │  portfolio      │   │ notification   │      │  Kafka events       │
   │ positions/P&L   │   │  WS pushes     │      │  (OrderExecuted,    │
   └────────────────┘   └────────────────┘      │   PositionUpdated…)  │
                                                └──────────┬─────────┘
      PostgreSQL (per-service schemas) ◄─── Redis (cache/rate limit) ◄┘
```

### 3.2 Final evolution target

```
STOCKFORGE
    │
    ├── Trading Platform          Engineering Platform
    │        │                          │
    │        ▼                          ├── CI/CD (GitHub Actions + Jenkins)
    │   Web / APIs                     └── Performance
    │        │
    │        ▼
    │   Microservices
    │        │
    └────────┴──────┬──────────────┘
                   ▼
             Quality Gates
                   │
                   ▼
              Containers
                   │
                   ▼
              Kubernetes
                   │
                   ▼
                 Cloud
                   │
                   ▼
             Observability
                   │
                   ▼
        Production Operations
                   │
                   ▼
         HFT Evolution Phase
```

### 3.3 CI/CD architecture (two first-class systems)

```
Developer → GitHub PR → GitHub Actions (lint/unit/integration/contract/security/build/container)
    → Artifact/Container Registry
    → Jenkins (environment deployment / system integration / performance smoke / load / baseline / gates)
    → Kubernetes → Staging → Validation → Production
```

**Status:** `APPROVED`

---

## 4. Repositories & Local Layout (ONE SERVICE = ONE FOLDER = ONE REPO)

GitHub Organization: **Stock-Forge** (org URL: https://github.com/Stock-Forge)

Local convention on both devices — every repo is an **independent folder with its own `.git`** and maps 1:1 to a GitHub repo in the org. Folders are created **on demand** when their phase starts.

```
C:\CODE\HFT Application\            (or any agreed root — must be identical on both devices)
│
├── stockforge-project-context\      ← EXISTS, separate git repo
├── stockforge-web\                  ← separate git repo (IMPLEMENTED, Day 4)
├── stockforge-api\                  ← separate git repo (IMPLEMENTED, Day 5)
├── stockforge-auth\                 ← separate git repo (Phase 3, IMPLEMENTED)
├── stockforge-order-service\        ← separate git repo (Phase 4)
├── stockforge-risk-service\         ← separate git repo (Phase 5)
├── stockforge-matching-engine\      ← separate git repo (Phase 6)
├── stockforge-market-data\          ← separate git repo (Phase 7)
├── stockforge-portfolio\            ← separate git repo (Phase 8)
├── stockforge-notification\         ← separate git repo (Phase 8)
├── stockforge-contracts\            ← separate git repo (Phase 1)
├── stockforge-performance\          ← separate git repo (Phase 16)
├── stockforge-infrastructure\       ← separate git repo (Phase 19/23)
├── stockforge-deployment\           ← separate git repo (Phase 18/19)
└── stockforge-shared-actions\       ← separate git repo (Phase 14)
```

| Repository | Responsibility | Status |
|---|---|---|
| stockforge-project-context | Project state, context, prompts, ADRs | `IN PROGRESS` |
| stockforge-web | Trading UI (login, dashboard, market data, order entry, book, portfolio, P&L, history, WebSocket) | `IMPLEMENTED` (scaffold: Vite + React + TS shell, Day 4) |
| stockforge-api | Client-facing API (routing, validation, auth integration, rate limiting, correlation IDs, docs) | `IMPLEMENTED` (scaffold: Spring Boot, /api/health, structured logging, correlation-ID filter, Day 5) |
| stockforge-auth | Registration, login, logout, hashing, auth, authz, roles, tokens | `IMPLEMENTED` (register/login bcrypt + JWT, Day 6) |
| stockforge-order-service | Create/validate/cancel orders, lifecycle, state, events | `PLANNED` |
| stockforge-risk-service | Balance/margin/position/quantity limits, risk rules, rejection | `PLANNED` |
| stockforge-matching-engine | Order book, price-time priority, matching, fills, executions | `PLANNED` |
| stockforge-market-data | Simulated market, price gen, bid/ask, LTP, volume, depth, WS | `PLANNED` |
| stockforge-portfolio | Balances, positions, holdings, P&L, trade history | `PLANNED` |
| stockforge-notification | Order/execution/system notifications | `PLANNED` |
| stockforge-contracts | OpenAPI, API contracts, Kafka schemas, message schemas | `IMPLEMENTED` (foundation: OpenAPI v1.0.0 + 7 events) |
| stockforge-performance | Scenarios (smoke/load/stress/soak), tests, data, baselines, thresholds, reports, regression logic | `PLANNED` |
| stockforge-infrastructure | IaC (local → Docker → Compose → local K8s → AWS: VPC/EKS/RDS/Redis/Kafka/ALB/ECR/monitoring) | `PLANNED` |
| stockforge-deployment | K8s manifests, Helm charts, env config, version pinning per environment | `PLANNED` |
| stockforge-shared-actions | Reusable CI/CD: test, build, security, container, performance, artifact, deployment | `PLANNED` |

**Rule:** create repos progressively, only when their phase requires them. Explain each before creating. Never nest a repo inside another repo's folder.

**Status:** `APPROVED`

---

## 5. Proposed Technology Stack

| Layer | Choice | Why | Alternative |
|---|---|---|---|
| Backend | **Java 21 LTS + Spring Boot 4.x (4.1.0) + Maven** | Trading-industry standard; JVM gives us the HFT-relevant perf work later (GC, JFR, async-profiler, object pooling, lock contention). Huge ecosystem and docs. Boot moved 3.x→4.x (start.spring.io is 4.x-only, Day 5). | Go (simpler goroutines, low alloc) — better only if we drop the JVM perf journey |
| Frontend | **React 18 + TypeScript + Vite** | Huge ecosystem, realistic production stack; Vite is fast and simple to teach. | Vue, Svelte |
| Realtime | **WebSocket** (browser-native) | Market data + notifications streaming. | SSE, polling |
| Database | **PostgreSQL 16** | Relational core, teaches schema/indexes/transactions/isolation/locks/pools/migrations. | MySQL |
| Cache | **Redis 7** | Caching, rate limiting, hot data, sessions; measurable perf win vs Postgres-only. | Memcached |
| Messaging | **Apache Kafka 3.x** (KRaft, single node locally via Docker) | Events, ordering, partitions, consumer groups, lag, DLQ — core trading architecture. | Redpanda, RabbitMQ |
| Container | **Docker + Docker Compose** | Local distributed execution (Phase 12-13). | Podman |
| Kubernetes | **kind** locally → **EKS** later | Real K8s (Deployment/Service/Probes/HPA/Ingress) without cloud cost. | minikube, k3s |
| CI/CD | **GitHub Actions** (PR/developer CI) + **Jenkins** (enterprise CI/CD: deploy, perf gates) | Both first-class; equivalent pipelines in both. | GitLab CI, CircleCI |
| Perf testing | **k6** (primary) — **JMeter/Gatling** as alternatives | Scriptable, CI-friendly, good histograms; later compare with JVM-based tools. | Locust, vegeta |
| Observability | **Prometheus + Grafana + OpenTelemetry**; **Loki** logs; **New Relic** later | Metrics/logs/traces + SLOs; industry standard, free to run locally. | Datadog (paid) |
| IaC | **Terraform** + **Helm** | Cloud + K8s as code (Phase 19/23). | Pulumi, Ansible |
| Perf analysis | **JFR/JMC**, async-profiler, GC logs, JVM flags | The Performance Engineer's native toolkit. | VisualVM, MAT |

**Status:** `APPROVED`

---

## 6. Service Boundaries

Identified service set (see §4). **Start-small principle (§42 of master prompt):** do not build all at once. Order and split rationale:

| Service | Why it exists | Why separate | Production equivalent |
|---|---|---|---|
| web | UI | Teams/roles differ; deploys independently | Trading frontend |
| api | One client-facing entry; routing/validation/rate-limit/correlation | Stable external contract; hides internals | BFF / API gateway |
| auth | Credentials + tokens | Security isolation; zero-trust zone | IAM / auth service |
| order-service | Order lifecycle & state | Core domain; own DB tables | Order management system |
| risk-service | Limits, margin, position checks | Independent enforcement & testing | Risk engine |
| matching-engine | Order book + matching | Performance-critical; isolated tuning | Matching engine (exchange) |
| market-data | Price simulation + fanout | High-volume streaming | Market data feed |
| portfolio | Positions, P&L, history | Own domain; reporting | P&L/positions service |
| notification | Order/execution/system pushes | Async; independent scaling | Notification service |

**Not decided yet (defer to avoid overengineering):** whether notification starts as a thin service inside api and splits later; whether portfolio/risk share a DB initially. Final call is part of approval.

**Status:** `APPROVED`

---

## 7. API Catalogue & Critical Traversals

### 7.1 Catalogue (initial)

| Method | Path | Service | Notes |
|---|---|---|---|
| POST | /api/auth/register | auth | create user |
| POST | /api/auth/login | auth | returns token |
| POST | /api/auth/logout | auth | invalidate token |
| GET | /api/marketdata/{symbol} | market-data | last price, depth |
| GET | /ws/marketdata | market-data | streaming prices |
| POST | /api/orders | order-service | market / limit |
| GET | /api/orders | order-service | list |
| GET | /api/orders/{id} | order-service | detail + status |
| DELETE | /api/orders/{id} | order-service | cancel |
| GET | /api/portfolio | portfolio | balances |
| GET | /api/portfolio/positions | portfolio | holdings |
| GET | /api/portfolio/pl | portfolio | P&L |
| GET | /api/orders/history | order-service | history |
| GET | /ws/notifications | notification | live pushes |

### 7.2 Critical traversal — POST /api/orders (full path, documented in every session)

```
Browser
 ↓ REST (Bearer token, correlation ID)
api  → auth check → validate payload → rate limit (Redis)
 ↓  → order-service (create order, status ACCEPTED)
        → risk-service (balance/limit checks → PASS/REJECT)
        → order-service → publish OrderAccepted / OrderRejected (Kafka)
        → matching-engine (match against order book)
             → fills (partial/full) → publish OrderExecuted (Kafka)
        → portfolio (positions/P&L update → PositionUpdated)
        → notification (WS push to browser)
        → api → HTTP response (OrderAccepted with id)
```

Document for every traversal: request, auth, validation, service calls, DB calls, cache calls, events, response, latency contributors, timeout/retry/failure, idempotency, logging, metrics, tracing.

**Status:** `APPROVED`

---

## 8. Events (Kafka)

| Event | Producer | Consumers | Purpose |
|---|---|---|---|
| OrderCreated | order-service | matching-engine, notification | order entered |
| OrderAccepted | order-service | notification | validation passed |
| OrderRejected | order-service (or risk-service) | notification, api | risk/validation failure |
| OrderExecuted | matching-engine | portfolio, notification, order-service | fill(s) happened |
| OrderCancelled | order-service | matching-engine, notification | order cancelled |
| PositionUpdated | portfolio | notification, api | positions changed |
| MarketPriceUpdated | market-data | matching-engine, api | price tick (high volume) |

Per event document: producer, topic, partitioning key, consumer group, ordering needs, failure handling (retries, idempotency, DLQ).

**Status:** `APPROVED`

---

## 9. Data Layer

### 9.1 PostgreSQL — initial schema areas (per service, own tables)

- auth: `users`, `sessions`/tokens
- order-service: `orders` (id, symbol, side, type, price, qty, status, filledQty, timestamps)
- portfolio: `balances`, `positions`, `executions`
- risk-service: `limits`, `risk_rules`
- market-data: `instruments`, recent prices (avoid hot-write; WS streams most)

Teach: schema, indexes, transactions, isolation, locks, connection pools, migrations (Flyway/Liquibase), query optimization. Intentionally reproduce: slow query, missing index, pool exhaustion, lock contention, deadlock, long transaction — **measure before fixing**.

**Initial stance:** one PostgreSQL instance, one database per service (or schema separation) locally; split to separate instances when Kafka/K8s arrives. Decide in Phase 9.

### 9.2 Redis — uses

Caching (instruments, hot market data), rate limiting, session/token store, hot data. Compare **PostgreSQL-only vs PostgreSQL+Redis** with measurements.

**Status:** `APPROVED`

---

## 10. Docker & Kubernetes

- **Docker:** Dockerfiles (explained line-by-line), images, layers, networking, volumes, env vars, health checks, resource limits. Compose for local distributed execution.
- **Kubernetes:** Pod, Deployment, Service, ConfigMap, Secret, Ingress, readiness/liveness probes, resource requests/limits, HPA, rolling deployments. Intentional failures: CrashLoopBackOff, failed readiness, bad config, resource exhaustion, failed deployment.

**Status:** `APPROVED`

---

## 11. CI/CD

### 11.1 Division of labor (decision, §12 of master prompt)

- **GitHub Actions (Developer CI):** lint, unit, integration, contract, security scan, build, container image — on PRs and push. Repository-native, fast feedback.
- **Jenkins (Enterprise CI/CD):** deploy to test environment, system integration/E2E, performance smoke, load tests, baseline comparison, performance gates, promotion. Why: matches how enterprises separate developer CI from release/quality engineering, and gives us agent architecture, pipelines, credentials, shared libraries, scheduled/parameterized jobs.

### 11.2 Test frequency

- Every commit: fast tests
- PR: functional + lightweight perf checks
- Merge to main: build + integration
- Nightly/scheduled: load, stress, soak, capacity
- Before production: full production-like validation

### 11.3 Pipeline shapes

- Service CI: commit → lint → unit → integration → contract → build → security → image
- System CI: deploy test env → integration → E2E → perf validation → PASS/FAIL
- Performance CI: deploy → perf smoke → metrics → baseline compare → gate → PASS/FAIL

**Status:** `APPROVED`

---

## 12. Performance Strategy

- Baselines, thresholds, reports, regression logic in `stockforge-performance`.
- Perf gates with tolerance (no false failures). Example: baseline Order API p95 = 150 ms → new p95 = 580 ms = FAIL.
- Metrics: orders/sec, matches/sec, p50/p95/p99, order-to-execution latency, market-data delivery latency, CPU, memory.
- Connect: application → instrumentation → metrics/logs/traces → perf test → CI/CD → SLO → production monitoring.
- HFT evolution: every optimization = baseline → hypothesis → change → measurement → result → conclusion.

**Status:** `APPROVED`

---

## 13. Observability & SLOs

- Structured logs, metrics, traces; correlation/request/trace IDs; New Relic or equivalent later; Prometheus/Grafana/OTel locally.
- SLO/SLI candidates: availability, latency (API p95), error rate, throughput, order-to-execution latency, market-data delivery latency. Concepts: SLI, SLO, SLA, error budget.
- Rule: measure baseline first; never invent arbitrary targets.

**Status:** `APPROVED`

---

## 14. Security

Progressive: password hashing (bcrypt/argon2), auth/authz (JWT + roles), secrets, HTTPS, input validation, rate limiting, dependency/container scanning, least privilege, audit logging, secure CI/CD.

**Status:** `APPROVED`

---

## 15. Infrastructure & Environments

- Progression: local app → Docker → Compose → kind → AWS (VPC, EKS, RDS, Redis, Kafka, ALB, ECR, monitoring).
- Environments: development / staging / production with pinned image versions (in `stockforge-deployment`).
- Production patterns: rolling, canary, blue-green.

**Status:** `APPROVED`

---

## 16. Incident Engineering

Intentional incidents: DB slowdown, connection exhaustion, Redis failure, Kafka lag, CPU saturation, memory leak, latency spike, service crash, bad deployment, config error, performance regression. Each documented: Symptoms → Detection → Investigation → Root cause → Mitigation → Recovery → Permanent fix → Prevention.

**Status:** `APPROVED`

---

## 17. HFT Evolution (Phase 2)

Evolve toward: high throughput, low latency, order-book optimization, concurrency, lock contention, memory allocation, GC, object pooling, CPU affinity, thread models, lock-free structures, serialization, binary protocols, high-performance messaging, market-data fanout, deterministic processing, latency histograms, ns/µs measurement, hardware-aware testing, NUMA.

**Explicit position:** initial microservice architecture is NOT a real exchange-grade HFT system; we progressively explain why HFT architecture looks different. Every change measured (baseline → hypothesis → change → measurement → result → conclusion).

**Hot-path vs control-plane split (2026-08-10, see ADR 0003, PROPOSED):** the latency-critical
trading core (matching engine, market data, strategy, risk, OMS, exchange gateway) is built in
**plain Java, not Spring Boot** — the JVM's own perf engineering (GC, JFR, JIT, locks, lock-free,
µs/ns measurement) is the learning objective, and a C++ equivalent can be compared later. Spring
Boot, PostgreSQL, Redis, Kafka stay in the control plane (REST/admin/monitoring/reporting), off the
hot path. First practical step: a small Java app with a controlled allocation/contention problem
recorded with **JFR** to observe GC/CPU/lock behavior.

**Status:** `APPROVED`

---

## 18. Development Phases & Repository Creation Order

```
Phase 0    Architecture and planning            <- CURRENT (this doc, awaiting approval)
Phase 1    Git/GitHub org + repo foundation     -> stockforge-project-context (DONE), contracts
Phase 2    Basic application and UI             -> web, api
Phase 3    Authentication                       -> auth
Phase 4    Order management                     -> order-service
Phase 5    Risk management                      -> risk-service
Phase 6    Matching engine                      -> matching-engine
Phase 7    Market data                          -> market-data
Phase 8    Portfolio/P&L + notifications        -> portfolio, notification
Phase 9    Database optimization
Phase 10   Redis
Phase 11   Kafka/event-driven architecture
Phase 12   Docker
Phase 13   Docker Compose
Phase 14   CI with GitHub Actions               -> shared-actions
Phase 15   Jenkins
Phase 16   Performance testing                  -> performance
Phase 17   Performance regression gates
Phase 18   Kubernetes                           -> deployment
Phase 19   Deployment automation                -> infrastructure
Phase 20   Observability
Phase 21   SLO/SRE
Phase 22   Security hardening
Phase 23   Production-like cloud infrastructure
Phase 24   Failure injection and incident response
Phase 25   Production rollout simulation
Phase 26   HFT evolution
```

Repo creation order (progressive, each with explanation before creation):

1. `stockforge-project-context` ✅ (exists)
2. `stockforge-contracts` ✅ (Phase 1 — contracts/schemas first so services don't break each other)
3. `stockforge-web` + `stockforge-api` ✅ (Phase 2)
4. `stockforge-auth` ✅ (Phase 3 — Day 6 register/login; Day 7 roles + JWT verification)
5. `stockforge-order-service` (Phase 4)
6. `stockforge-risk-service` (Phase 5)
7. `stockforge-matching-engine` (Phase 6)
8. `stockforge-market-data` (Phase 7)
9. `stockforge-portfolio` + `stockforge-notification` (Phase 8)
10. `stockforge-shared-actions` (Phase 14)
11. `stockforge-deployment` (Phase 18)
12. `stockforge-infrastructure` (Phase 19/23)
13. `stockforge-performance` (Phase 16 — can start earlier if desired)

**Status:** `APPROVED`

---

## 19. Two-Device Git Workflow

- GitHub is the source of truth. Session continuity lives in this repo, not AI memory.
- Before work: `git pull`, inspect `git status` / `git branch` / `git log -5`.
- End of day: stop clean → tests → update README/state/prompts → review diff → commit → push → verify.
- Conflicts/unexpected changes: STOP, explain, never destructive.
- Prompts themselves are version-controlled artifacts.

**Status:** `IMPLEMENTED` (workflow defined and enforced)

---

## 20. Risks & Trade-offs

| Risk | Mitigation |
|---|---|
| Scope too broad for a beginner | 30-min units, progressive phases, one technology per phase |
| JVM/Spring complexity early | Start minimal; teach incrementally; thin services first |
| Ten microservices = overengineering | Start-small principle; split only with reason (§42) |
| Two-device merge conflicts | Pull-first discipline; one active device at a time; small commits |
| Performance-gate flakiness | Tolerances + baseline management; never false-fail |
| Simulator ≠ HFT confusion | Explicit framing: V1 is production-style, HFT evolution is later, measured |
| Skill gap in testing | Teach testing alongside each service; no skips |
| GitHub org/repo naming collisions | Verify `StockForge` availability first; fallback name agreed |

**Status:** `APPROVED`

---

## 21. 30-Minute Learning Roadmap

```
Day 0  ✅ Project-context foundation (this repo)                            [done]
Day 1  ✅ Phase 0 architecture proposal + approval                          [done]
Day 1.5✅ Approval recorded, day-by-day guide written, org created/pushed   [done]
Day 2    ✅ GitHub org + repo + git workflow practice (push done; clone on device B)
Day 3    ✅ stockforge-contracts — OpenAPI + contract thinking                       [done]
Day 4    ✅ stockforge-web scaffold (Vite + React + TS)                              [done]
Day 5    ✅ stockforge-api scaffold (Spring Boot, /api/health, logs, correlation ID)       [done]
Day 6    ✅ auth — register/login (bcrypt + JWT) Part 1                             [done]
Day 7    auth — roles + JWT verification (@Authenticated, /api/auth/me) Part 2
Day 8    order-service — create/list/cancel orders (in-memory first)
Day 9    order-service + risk checks (mock risk)
Day 10   PostgreSQL — schema, migrations, index lesson
Day 11   risk-service — limits, margin, position checks
Day 12   matching-engine — order book + price-time priority (in-memory)
Day 13   matching-engine — partial/full fills + execution events
Day 14   market-data — simulated prices + WebSocket fanout
Day 15   portfolio — positions, P&L, history
Day 16   notification — WebSocket order/execution pushes
Day 17   Integration — all services talk end-to-end (POST /orders traversal)
Day 18   DB optimization — connection pools, locks, deadlocks (measured)
Day 19   Redis — caching + rate limiting vs Postgres-only (measured)
Day 20   Kafka — events, consumer groups, idempotency, lag
Day 21   Kafka failure lab — duplication, DLQ, lag investigation
Day 22   Docker — Dockerfiles explained, every service containerized
Day 23   Docker Compose — full platform locally
Day 24   GitHub Actions — PR checks + reusable workflows (shared-actions)
Day 25   Branch protection + required checks
Day 26   Jenkins — controller/agents + first Jenkinsfile
Day 27   Jenkins — shared library, credentials, scheduled jobs
Day 28   Jenkins — performance pipeline (deploy → smoke → load → gate)
Day 29   Performance testing — k6 scenarios + baselines
Day 30   Performance regression gates in CI/CD
Day 31   Kubernetes (kind) — Pod/Deployment/Service/Probes
Day 32   Kubernetes — ConfigMap/Secret/Ingress/HPA/rolling deploys
Day 33   Deployment repo — pinned versions per environment
Day 34   Observability — Prometheus/Grafana/OTel + SLOs
Day 35   Security hardening + scanning
Day 36   Cloud — AWS/EKS/IaC (production-style)
Day 37   Failure injection + incident response
Day 38   Production rollout simulation (canary/blue-green/rollback)
Day 39+  HFT evolution — measured optimizations (see §17)
Day 40   Platform study — deep-dive how a REAL platform (Zerodha Kite, Groww, Coinbase)
         implements today's domain (order book / matching / market data / risk); deliverable:
         written comparison of their design vs ours (see §25 habit 6)
Day 41   Incident case study — take one real issue from ISSUES_LOG and write a full
         SRE-style post-mortem: timeline, detection, investigation, root cause, mitigation,
         prevention (see §25 habit 7)
Day 42   Teach someone else — record a 10-minute explainer / write a blog post / teach a
         friend one concept we built, in plain language (see §25 habit 9)
Day 43   Monthly code review — refactor one component from weeks ago; write down what's
         better now and why (see §25 habit 10)
```

Recurring habits run EVERY day/week (see §25): teach-back each evening, hypothesis before
every change, build-the-primitive side quests weekly, real-platform reading weekly, and
the three dedicated days above (40-42) revisit these practices deeply.

Roadmap is reviewed at each phase boundary and adjusted after feedback. Detailed manual steps for every day live in `DAY_BY_DAY_GUIDE.md` (detail is added when the day is reached).

**Status:** `APPROVED`

---

## 22. Architecture Decision Records

Stored in `project-context/adr/`. One file per decision: Context, Decision, Alternatives, Consequences.

| ADR | Topic | Status |
|---|---|---|
| 0001 | GitHub org + one-folder-one-repo-per-service layout + two-device workflow | `ACCEPTED` (2026-08-06) |
| 0002 | Technology stack (Java 21 / Spring Boot / React / PostgreSQL / Redis / Kafka / k6 / Prometheus…) | `ACCEPTED` (2026-08-06) |
| … | PostgreSQL, Redis, Kafka, service boundaries, deployment repo, Jenkins, GitHub Actions, Kubernetes, performance repo, HFT changes | `PLANNED` (create when decided) |

**Status:** `APPROVED`

---

## 23. Key Rules

- Do NOT overengineer; smallest architecture that teaches the concept; split with a reason.
- Teach before automating; explain every important line of generated artifacts.
- 30-minute days; clean stopping points; incomplete work recorded openly (never hidden).
- Production realism: Local vs Production vs Difference for every feature.
- Never document something as implemented when it is not.
- GitHub Actions + Jenkins both first-class; create equivalent pipelines in both.
- **Phase 0 architecture must be approved before implementation begins.**
- The 10 "0.1%" practices in §25 are part of the project, not optional extras — they are
  enforced daily in `START_OF_DAY.md` and on dedicated roadmap days (40-43).
- **Explain hard concepts "cheat-sheet style" (user preference).** When teaching a difficult topic
  (e.g. GC, JFR, lock contention, React re-renders, JWT), explain it like the cheat sheets in
  `project-context/cheatsheets/`: start with the WHOLE PICTURE in one picture (Mermaid diagram),
  then Key Findings (the take-aways in plain words), map unfamiliar concepts to familiar ones
  (backend pillars ↔ frontend mirrors), and end with a practical "plant the bug → hunt it" lab plan.
  Simplify before automating; a diagram + one-sentence idea beats a paragraph of jargon.

---

## 24. README

Every repository must have a README (how to run, architecture, APIs, dependencies, tests, Docker, CI/CD, limitations). This repo's README: `stockforge-project-context/README.md`.

**Status:** `IMPLEMENTED`

---

## 25. The 10 "0.1%" Engineering Practices

> Approved 2026-08-07. These ten habits are the difference between "collecting tools"
> and *distilling understanding*. They are baked into the daily prompts (`START_OF_DAY.md`,
> `SESSION_PROMPTS.md`), the issues log, the guide, and dedicated roadmap days 40-43.
> "The project is complete only when the habit is done, not when the code compiles."

| # | Practice | Where it lives in the project |
|---|---|---|
| 1 | **Teach-back (Feynman).** After each day, write the concept in your own words as if to a smart friend. If you can't explain it simply, you don't understand it. | `START_OF_DAY.md` Phase 3 (teach-back step); written into `JOURNEY_SO_FAR.md` |
| 2 | **Hypothesis before change.** State "I believe X happens because of Y" before ANY change; then measure, change, re-measure. No guessing. | `START_OF_DAY.md` Phase 2; extends §12 beyond performance to every change |
| 3 | **Deep, not wide.** Every tool is a window into a fundamental: Kafka → partitioning/ordering/exactly-once; JVM → memory model/GC/JIT; order book → price-time priority. Ask "what problem does this solve, what breaks if it's gone?" | `START_OF_DAY.md` Phase 2; `DAY_BY_DAY_GUIDE.md` production sections |
| 4 | **Deliberately build it wrong.** Remove an index, exhaust a pool, kill a service mid-order, watch Kafka redeliver — then fix. Engineers who've seen systems break know more. | Phase 24 + expanded failure list in `DAY_BY_DAY_GUIDE.md`; issues logged in `ISSUES_LOG.md` |
| 5 | **Build the primitive.** Weekly side quests: write your own JWT, token-bucket rate limiter, mini order book, latency histogram — then understand the production library 10x better. | New §26 "Primitive side quests" + weekly rhythm in `START_OF_DAY.md` |
| 6 | **Read how the real platforms do it.** Weekly reading of Zerodha (zerodha.tech), Groww, Coinbase, exchange white papers, open-source trading systems. Compare: "they do X with 100 engineers + 10 DCs, we do it in 100 lines — where's the difference?" | Dedicated **Day 40**; weekly reading habit in `START_OF_DAY.md` |
| 7 | **Every bug is a case study.** Issues logged SRE-style: timeline, detection, investigation, root cause, mitigation, prevention. A written body of "why things fail" is senior-engineer material. | `ISSUES_LOG.md` (upgraded template); dedicated **Day 41** |
| 8 | **Observability from day one.** Wire metrics/logs/traces + correlation IDs into every service; a perf engineer who can prove a 40% regression with a flame graph is top-0.1% material. | §13 + enforced from Day 5 (`/api/health`, logging, correlation ID) |
| 9 | **Teach someone else.** Explain days to a friend, write a blog post, make the README great. Teaching forces precision and compounds the career. | Dedicated **Day 42**; READMEs required (§24) |
| 10 | **Review your own old code monthly.** Refactor a component from weeks ago; document what's better now and why. This is where taste develops. | Dedicated **Day 43**; monthly rhythm in `START_OF_DAY.md` |

**Status:** `APPROVED`

---

## 26. Primitive Side Quests (weekly, habit 5)

Build a toy version of something you're using, from scratch, in any language. One per
week is enough. Suggested list (add more as the project grows):

- JWT — sign + verify by hand (understand the HMAC/RS256 algorithm, claims, expiry)
- Token-bucket rate limiter (the idea behind Redis rate limiting, Phase 10)
- Mini order book + price-time matching engine (the heart of the matching-engine phase)
- Latency histogram (p50/p95/p99 — the tool every perf engineer must own)
- Connection pool (the idea behind pool exhaustion failures, Phase 9/24)
- Kafka-like log: append-only file with offset + consumer reading from a position
- Mini WebSocket fanout (the idea behind market-data streaming)
- LRU cache (the idea behind Redis caching)

**Status:** `APPROVED`

**Status of §24-26:** `APPROVED` (2026-08-07)
