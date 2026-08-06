# PROJECT_CONTEXT.md

> Permanent architecture/context document for **StockForge**.
> A completely new AI must understand the whole project from this file.
> Keep this file up to date. Never let it become stale.
>
> Status legend: `PLANNED` | `IMPLEMENTED` | `IN PROGRESS` | `DEPRECATED`

---

## 1. Project Purpose

Build **StockForge** — a production-style stock trading platform and engineering laboratory. The project is a long-term learning and engineering program for someone who is a Performance Engineer but relatively new to software development, backend/frontend, Git/GitHub, CI/CD, Jenkins, Docker, Kubernetes, cloud, IaC, and distributed systems.

The project deliberately teaches **WHY** each technology exists, not just its commands.

**Status:** `IMPLEMENTED` (context documented) / project itself `PLANNED`

---

## 2. Final Goal

A complete production-grade stock trading platform, understood end-to-end (request/event/service/infrastructure lifecycle), automated delivery and performance validation through **both GitHub Actions and Jenkins**, deployed and operated on Kubernetes, with SLOs and observability, deliberate production-failure injection/resolution — then progressive evolution toward **high-throughput / low-latency HFT-style architecture**.

**Status:** `PLANNED`

---

## 3. Target Architecture

### 3.1 Final evolution target

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

**Status:** `PLANNED`

### 3.2 CI/CD architecture (two first-class systems)

```
Developer → GitHub PR → GitHub Actions (lint/unit/integration/contract/security/build/container)
    → Artifact/Container Registry
    → Jenkins (environment deployment / system integration / performance smoke / load / baseline / gates)
    → Kubernetes → Staging → Validation → Production
```

**Status:** `PLANNED`

---

## 4. Repositories

GitHub Organization: **StockForge**

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

**Rule:** create repositories progressively, only when their phase requires them. Explain each before creating.

---

## 5. Services & Boundaries

Identified service set per section 4 (`stockforge-web`, `stockforge-api`, `stockforge-auth`, `stockforge-order-service`, `stockforge-risk-service`, `stockforge-matching-engine`, `stockforge-market-data`, `stockforge-portfolio`, `stockforge-notification`).

**Boundary decisions (initial rationale):**

- Matching engine is deliberately separate and performance-critical (order book, price-time priority, matching, fills).
- Risk service separated so order validation can be independently enforced and tested.
- Contracts separate to prevent silent contract breakage between repos.
- Performance repo separate because it tests the whole platform, not a service.
- Deployment repo separate because it answers *"what version of every service is in every environment"*.

Full service-boundary analysis is part of **Phase 0** and requires approval.

**Status:** `PLANNED` (preliminary list, pending Phase 0 approval)

---

## 6. APIs

API catalogue, critical API traversals (e.g. `POST /orders` full path: Browser → LB → API → Auth → Validation → Order Service → Risk Service → Redis → PostgreSQL → Kafka → Matching Engine → Execution → Portfolio → Kafka → WebSocket → Browser), and user journeys (registration, login, market data, market/limit orders, cancel, partial/full execution, history, portfolio, positions, P&L, logout) are **Phase 0** deliverables.

**Status:** `PLANNED`

---

## 7. Events (Kafka)

Target events: `OrderCreated`, `OrderAccepted`, `OrderRejected`, `OrderExecuted`, `OrderCancelled`, `PositionUpdated`, `MarketPriceUpdated`.

For each event document: producer, topic, partitioning considerations, consumer, downstream effects, failure handling.

**Status:** `PLANNED`

---

## 8. Data Layer

- **PostgreSQL:** schema, indexes, transactions, isolation, locks, connection pools, migrations, query optimization. Intentional failure reproduction (slow query, missing index, pool exhaustion, lock contention, deadlock, long tx) — measure before fixing.
- **Redis:** caching, rate limiting, hot data, sessions; measure PostgreSQL-only vs PostgreSQL+Redis.

**Status:** `PLANNED`

---

## 9. Docker & Kubernetes

- **Docker:** Dockerfiles (explained line-by-line), images, layers, networking, volumes, env vars, health checks, resource limits. Docker Compose for local distributed execution.
- **Kubernetes:** Pod, Deployment, Service, ConfigMap, Secret, Ingress, readiness/liveness probes, resource requests/limits, HPA, rolling deployments. Intentional failures: CrashLoopBackOff, failed readiness, bad config, resource exhaustion, failed deployment.

**Status:** `PLANNED`

---

## 10. CI/CD

- **GitHub Actions:** PR checks, repository-native CI, reusable workflows, artifacts, secrets, environments, branch protection, required checks.
- **Jenkins (first-class):** controller/agents, Jenkinsfile, pipeline stages, credentials, shared libraries, scheduled/parameterized jobs, distributed execution, enterprise integration, performance test orchestration.
- **Service CI:** per-repo lint → unit → integration → contract → build → security → Docker image.
- **System CI:** deploy test env → integration → E2E → performance validation → PASS/FAIL.
- **Performance CI/CD:** deploy → perf smoke → metrics → baseline compare → gate → PASS/FAIL.
- **Test frequency:** every commit (fast), PR (functional + light perf), main (build + integration), nightly (load/stress/soak/capacity), pre-prod (full validation).

**Status:** `PLANNED`

---

## 11. Performance Strategy

- Baselines, thresholds, reports, regression logic in `stockforge-performance`.
- Perf gates with tolerance to avoid false failures.
- Metrics tracked: orders/sec, matches/sec, p50/p95/p99, order-to-execution latency, market-data delivery latency, CPU, memory.
- Connect: application → instrumentation → metrics/logs/traces → performance test → CI/CD → SLO → production monitoring.

**Status:** `PLANNED`

---

## 12. Observability & SLOs

- Observability: structured logs, metrics, traces, correlation/request/trace IDs; New Relic or equivalent later.
- SLO/SLI: availability, latency, error rate, throughput, order-to-execution latency, market-data delivery latency. SLI/SLO/SLA/error-budget concepts.
- Rule: measure baseline first; never invent arbitrary targets.

**Status:** `PLANNED`

---

## 13. Security

Progressive: password hashing, auth/authz, secrets, HTTPS, input validation, rate limiting, dependency/container scanning, least privilege, audit logging, secure CI/CD.

**Status:** `PLANNED`

---

## 14. Infrastructure & Environments

- Progression: local app → Docker → Docker Compose → local K8s → AWS (VPC, EKS, RDS, Redis, Kafka, ALB, ECR, monitoring).
- Environments: development / staging / production with pinned image versions.
- Production patterns: rolling, canary, blue-green.

**Status:** `PLANNED`

---

## 15. Incident Engineering

Intentional incidents: DB slowdown, connection exhaustion, Redis failure, Kafka lag, CPU saturation, memory leak, latency spike, service crash, bad deployment, config error, performance regression. Each documented: Symptoms → Detection → Investigation → Root cause → Mitigation → Recovery → Permanent fix → Prevention.

**Status:** `PLANNED`

---

## 16. HFT Evolution (Phase 2)

Evolve toward: high throughput, low latency, order-book optimization, concurrency, lock contention, memory allocation, GC, object pooling, CPU affinity, thread models, lock-free structures, serialization, binary protocols, high-performance messaging, market-data fanout, deterministic processing, latency histograms, ns/µs measurement, hardware-aware testing, NUMA.

Every optimization: baseline → hypothesis → change → measurement → result → conclusion.

**Explicit position:** the initial microservice architecture is NOT a real exchange-grade HFT system; we progressively explain why HFT architecture looks different.

**Status:** `PLANNED`

---

## 17. Development Phases

```
Phase 0    Architecture and planning          <- CURRENT
Phase 1    Git/GitHub organization and repository foundation
Phase 2    Basic application and UI
Phase 3    Authentication
Phase 4    Order management
Phase 5    Risk management
Phase 6    Matching engine
Phase 7    Market data
Phase 8    Portfolio/P&L
Phase 9    Database optimization
Phase 10   Redis
Phase 11   Kafka/event-driven architecture
Phase 12   Docker
Phase 13   Docker Compose
Phase 14   CI with GitHub Actions
Phase 15   Jenkins
Phase 16   Performance testing
Phase 17   Performance regression gates
Phase 18   Kubernetes
Phase 19   Deployment automation
Phase 20   Observability
Phase 21   SLO/SRE
Phase 22   Security hardening
Phase 23   Production-like cloud infrastructure
Phase 24   Failure injection and incident response
Phase 25   Production rollout simulation
Phase 26   HFT evolution
```

---

## 18. Two-Device Git Workflow

- GitHub is the source of truth. Session continuity lives in this repo, not in AI memory.
- Before work: `git pull`, inspect `git status` / `git branch` / `git log -5`.
- End of day: stop clean → tests → update README/state/prompts → review diff → commit → push → verify.
- Conflicts/unexpected changes: STOP, explain, never destructive.
- Prompts themselves are version-controlled artifacts.

**Status:** `IMPLEMENTED` (workflow defined and enforced)

---

## 19. Architecture Decision Records

Stored in `project-context/adr/`. One file per decision: Context, Decision, Alternatives, Consequences.

Planned ADRs (create as decisions are made): PostgreSQL, Redis, Kafka, service boundaries, separate repos, deployment repo, Jenkins, GitHub Actions, Kubernetes, performance repo, HFT changes, GitHub Actions + Jenkins division.

**Status:** none yet (created on first decision)

---

## 20. Key Rules

- Do NOT overengineer; smallest architecture that teaches the concept; split with a reason.
- Teach before automating; explain every important line of generated artifacts.
- 30-minute days; clean stopping points; incomplete work recorded openly (never hidden).
- Production realism: Local vs Production vs Difference for every feature.
- Never document something as implemented when it is not.
- GitHub Actions + Jenkins both first-class; create equivalent pipelines in both.
- Phase 0 architecture must be approved before implementation begins.

---

## 21. README

Every repository must have a README (how to run, architecture, APIs, dependencies, tests, Docker, CI/CD, limitations). This repository's README lives at `stockforge-project-context/README.md`.

**Status:** `PLANNED` (create with repo foundation)
