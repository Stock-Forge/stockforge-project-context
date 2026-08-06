# PROJECT_CONTEXT.md

> Permanent architecture/context document for **StockForge**.
> A completely new AI must understand the whole project from this file.
> Keep this file up to date. Never let it become stale.
>
> Status legend: `PLANNED` | `PROPOSED` | `IMPLEMENTED` | `IN PROGRESS` | `DEPRECATED`
> `PROPOSED` = written in Phase 0, awaiting user approval.

---

## 1. Project Purpose

Build **StockForge** — a production-style stock trading platform and engineering laboratory. Long-term learning and engineering program for someone who is a Performance Engineer but relatively new to software development, backend/frontend, Git/GitHub, CI/CD, Jenkins, Docker, Kubernetes, cloud, IaC, and distributed systems.

The project deliberately teaches **WHY** each technology exists, not just its commands.

**Status:** `IMPLEMENTED` (context documented) / project itself `PROPOSED`

---

## 2. Final Goal

A complete production-grade stock trading platform, understood end-to-end (request/event/service/infrastructure lifecycle), automated delivery and performance validation through **both GitHub Actions and Jenkins**, deployed and operated on Kubernetes, with SLOs and observability, deliberate production-failure injection/resolution — then progressive evolution toward **high-throughput / low-latency HFT-style architecture**.

**Status:** `PROPOSED`

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

**Status:** `PROPOSED`

---

## 4. Repositories & Local Layout (ONE SERVICE = ONE FOLDER = ONE REPO)

GitHub Organization: **StockForge**

Local convention on both devices — every repo is an **independent folder with its own `.git`** and maps 1:1 to a GitHub repo in the org. Folders are created **on demand** when their phase starts.

```
C:\CODE\HFT Application\            (or any agreed root — must be identical on both devices)
│
├── stockforge-project-context\      ← EXISTS, separate git repo
├── stockforge-web\                  ← separate git repo (Phase 2)
├── stockforge-api\                  ← separate git repo (Phase 2)
├── stockforge-auth\                 ← separate git repo (Phase 3)
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
| stockforge-web | Trading UI (login, dashboard, market data, order entry, book, portfolio, P&L, history, WebSocket) | `PLANNED` |
| stockforge-api | Client-facing API (routing, validation, auth integration, rate limiting, correlation IDs, docs) | `PLANNED` |
| stockforge-auth | Registration, login, logout, hashing, auth, authz, roles, tokens | `PLANNED` |
| stockforge-order-service | Create/validate/cancel orders, lifecycle, state, events | `PLANNED` |
| stockforge-risk-service | Balance/margin/position/quantity limits, risk rules, rejection | `PLANNED` |
| stockforge-matching-engine | Order book, price-time priority, matching, fills, executions | `PLANNED` |
| stockforge-market-data | Simulated market, price gen, bid/ask, LTP, volume, depth, WS | `PLANNED` |
| stockforge-portfolio | Balances, positions, holdings, P&L, trade history | `PLANNED` |
| stockforge-notification | Order/execution/system notifications | `PLANNED` |
| stockforge-contracts | OpenAPI, API contracts, Kafka schemas, message schemas | `PLANNED` |
| stockforge-performance | Scenarios (smoke/load/stress/soak), tests, data, baselines, thresholds, reports, regression logic | `PLANNED` |
| stockforge-infrastructure | IaC (local → Docker → Compose → local K8s → AWS: VPC/EKS/RDS/Redis/Kafka/ALB/ECR/monitoring) | `PLANNED` |
| stockforge-deployment | K8s manifests, Helm charts, env config, version pinning per environment | `PLANNED` |
| stockforge-shared-actions | Reusable CI/CD: test, build, security, container, performance, artifact, deployment | `PLANNED` |

**Rule:** create repos progressively, only when their phase requires them. Explain each before creating. Never nest a repo inside another repo's folder.

**Status:** `PROPOSED`

---

## 5. Proposed Technology Stack

| Layer | Choice | Why | Alternative |
|---|---|---|---|
| Backend | **Java 21 LTS + Spring Boot 3.x + Maven** | Trading-industry standard; JVM gives us the HFT-relevant perf work later (GC, JFR, async-profiler, object pooling, lock contention). Huge ecosystem and docs. | Go (simpler goroutines, low alloc) — better only if we drop the JVM perf journey |
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

**Status:** `PROPOSED` (needs approval — especially backend language)

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

**Status:** `PROPOSED`

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

**Status:** `PROPOSED`

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

**Status:** `PROPOSED`

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

**Status:** `PROPOSED`

---

## 10. Docker & Kubernetes

- **Docker:** Dockerfiles (explained line-by-line), images, layers, networking, volumes, env vars, health checks, resource limits. Compose for local distributed execution.
- **Kubernetes:** Pod, Deployment, Service, ConfigMap, Secret, Ingress, readiness/liveness probes, resource requests/limits, HPA, rolling deployments. Intentional failures: CrashLoopBackOff, failed readiness, bad config, resource exhaustion, failed deployment.

**Status:** `PROPOSED`

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

**Status:** `PROPOSED`

---

## 12. Performance Strategy

- Baselines, thresholds, reports, regression logic in `stockforge-performance`.
- Perf gates with tolerance (no false failures). Example: baseline Order API p95 = 150 ms → new p95 = 580 ms = FAIL.
- Metrics: orders/sec, matches/sec, p50/p95/p99, order-to-execution latency, market-data delivery latency, CPU, memory.
- Connect: application → instrumentation → metrics/logs/traces → perf test → CI/CD → SLO → production monitoring.
- HFT evolution: every optimization = baseline → hypothesis → change → measurement → result → conclusion.

**Status:** `PROPOSED`

---

## 13. Observability & SLOs

- Structured logs, metrics, traces; correlation/request/trace IDs; New Relic or equivalent later; Prometheus/Grafana/OTel locally.
- SLO/SLI candidates: availability, latency (API p95), error rate, throughput, order-to-execution latency, market-data delivery latency. Concepts: SLI, SLO, SLA, error budget.
- Rule: measure baseline first; never invent arbitrary targets.

**Status:** `PROPOSED`

---

## 14. Security

Progressive: password hashing (bcrypt/argon2), auth/authz (JWT + roles), secrets, HTTPS, input validation, rate limiting, dependency/container scanning, least privilege, audit logging, secure CI/CD.

**Status:** `PROPOSED`

---

## 15. Infrastructure & Environments

- Progression: local app → Docker → Compose → kind → AWS (VPC, EKS, RDS, Redis, Kafka, ALB, ECR, monitoring).
- Environments: development / staging / production with pinned image versions (in `stockforge-deployment`).
- Production patterns: rolling, canary, blue-green.

**Status:** `PROPOSED`

---

## 16. Incident Engineering

Intentional incidents: DB slowdown, connection exhaustion, Redis failure, Kafka lag, CPU saturation, memory leak, latency spike, service crash, bad deployment, config error, performance regression. Each documented: Symptoms → Detection → Investigation → Root cause → Mitigation → Recovery → Permanent fix → Prevention.

**Status:** `PROPOSED`

---

## 17. HFT Evolution (Phase 2)

Evolve toward: high throughput, low latency, order-book optimization, concurrency, lock contention, memory allocation, GC, object pooling, CPU affinity, thread models, lock-free structures, serialization, binary protocols, high-performance messaging, market-data fanout, deterministic processing, latency histograms, ns/µs measurement, hardware-aware testing, NUMA.

**Explicit position:** initial microservice architecture is NOT a real exchange-grade HFT system; we progressively explain why HFT architecture looks different. Every change measured (baseline → hypothesis → change → measurement → result → conclusion).

**Status:** `PROPOSED`

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
2. `stockforge-contracts` (Phase 1 — contracts/schemas first so services don't break each other)
3. `stockforge-web` + `stockforge-api` (Phase 2)
4. `stockforge-auth` (Phase 3)
5. `stockforge-order-service` (Phase 4)
6. `stockforge-risk-service` (Phase 5)
7. `stockforge-matching-engine` (Phase 6)
8. `stockforge-market-data` (Phase 7)
9. `stockforge-portfolio` + `stockforge-notification` (Phase 8)
10. `stockforge-shared-actions` (Phase 14)
11. `stockforge-deployment` (Phase 18)
12. `stockforge-infrastructure` (Phase 19/23)
13. `stockforge-performance` (Phase 16 — can start earlier if desired)

**Status:** `PROPOSED`

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

**Status:** `PROPOSED`

---

## 21. 30-Minute Learning Roadmap (first ~10 days)

```
Day 0  ✅ Project-context foundation (this repo)              [done]
Day 1  ✅ Phase 0 architecture proposal (this file)           [done — awaiting approval]
Day 2    GitHub org + repos + git workflow practice (push this repo, clone on device B)
Day 3    stockforge-contracts — OpenAPI + contract thinking
Day 4    stockforge-web scaffold (Vite + React + TS) + Day 4 README
Day 5    stockforge-api scaffold (Spring Boot, health, logs)
Day 6    auth — register/login (bcrypt + JWT) Part 1
Day 7    auth — authz + roles Part 2
Day 8    order-service — create/list/cancel orders (in-memory first)
Day 9    order-service + risk checks (mock risk)
Day 10   PostgreSQL — schema, migrations, index lesson
```

Longer roadmap continues through Phases 2-26 (see §18). Review and adjust after Phase 1 feedback.

**Status:** `PROPOSED`

---

## 22. Architecture Decision Records

Stored in `project-context/adr/`. One file per decision: Context, Decision, Alternatives, Consequences.

| ADR | Topic | Status |
|---|---|---|
| 0001 | GitHub org + one-folder-one-repo-per-service layout + two-device workflow | `PROPOSED` |
| 0002 | Technology stack (Java 21 / Spring Boot / React / PostgreSQL / Redis / Kafka / k6 / Prometheus…) | `PROPOSED` |
| … | PostgreSQL, Redis, Kafka, service boundaries, deployment repo, Jenkins, GitHub Actions, Kubernetes, performance repo, HFT changes | `PLANNED` (create when decided) |

**Status:** `PROPOSED`

---

## 23. Key Rules

- Do NOT overengineer; smallest architecture that teaches the concept; split with a reason.
- Teach before automating; explain every important line of generated artifacts.
- 30-minute days; clean stopping points; incomplete work recorded openly (never hidden).
- Production realism: Local vs Production vs Difference for every feature.
- Never document something as implemented when it is not.
- GitHub Actions + Jenkins both first-class; create equivalent pipelines in both.
- **Phase 0 architecture must be approved before implementation begins.**

---

## 24. README

Every repository must have a README (how to run, architecture, APIs, dependencies, tests, Docker, CI/CD, limitations). This repo's README: `stockforge-project-context/README.md`.

**Status:** `IMPLEMENTED`
