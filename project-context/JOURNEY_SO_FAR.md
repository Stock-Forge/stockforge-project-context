# STOCKFORGE — JOURNEY SO FAR

> A plain-language summary of everything we have established **up to just before
> creating the GitHub organization**. Read this to understand where the project stands
> and why it is shaped this way. No Git knowledge or commands required.
>
> This file is documentation only — it summarizes files that already exist in
> `stockforge-project-context\project-context\`.

---

## 0. The starting requirements (what you asked for)

The project began as a request to build a complete **stock trading platform** as a
long-term learning + engineering program. Over the discussion you refined it into
these concrete requirements:

1. **Name:** `StockForge` — not EquiFlow/EquiCore/TradeForge. "Stock" = the domain,
   "Forge" = an engineering/building/testing platform.
2. **Production-grade scope:** not a toy simulator. A realistic trading platform with
   UI, APIs, DB, Kafka, Redis, Docker, Kubernetes, CI/CD, observability.
3. **Jenkins is first-class:** learn BOTH GitHub Actions (developer CI / PR checks)
   and Jenkins (enterprise CI/CD / deployment + performance gates). Understand *why*
   enterprises use both.
4. **Phase 2 = HFT Evolution:** after V1 is complete, progressively evolve toward
   high-throughput / low-latency architecture. Every optimization must be measured
   (baseline → hypothesis → change → measurement → result → conclusion).
5. **30-minute days:** one project "day" = ~30 min of work; clean stopping points;
   incomplete work recorded, never hidden.
6. **Teach before automating:** you must understand WHAT/WHY/HOW, plus what production
   teams do, what can go wrong, how we detect it, how we fix it.
7. **Two devices, Git as source of truth:** work happens on two machines with different
   AI sessions. GitHub is the shared memory; state files + prompts are versioned.
8. **One service = one folder = one repo:** each component is its own repo, created
   progressively when its phase starts.
9. **You are the one doing GitHub creation manually**, guided step by step.

---

## 1. Day 0 — Project foundation (what exists on disk now)

A folder was created: `stockforge-project-context\project-context\` containing:

| File | Purpose |
|---|---|
| `MASTER_PROMPT.md` | The single prompt that drives the whole project (all rules, 47 sections). |
| `PROJECT_CONTEXT.md` | The permanent architecture/context document — a new AI reads this to understand everything. |
| `CURRENT_STATE.md` | Live progress state: phase, day, what's done/incomplete, next exact task. |
| `SESSION_PROMPTS.md` | Copy-paste resume prompt for a brand-new AI session + a record of each day. |
| `DAY_BY_DAY_GUIDE.md` | Your manual companion: GitHub setup steps, folder structure, day-by-day plan. |
| `adr/` | Architecture Decision Records (0001, 0002) — record *why* decisions were made. |
| `README.md` (repo root) | What this repo is and how to resume. |
| `CHANGELOG.md` (repo root) | One line per day of what changed. |

Rule that makes all of this work: **a brand-new AI session must never assume it
remembers anything** — it reads these files after `git pull` and reconstructs the
project.

---

## 2. The Master Prompt (the core agreement)

`MASTER_PROMPT.md` is the heart of the project. Key rules baked into it:

- You are my Senior Software Architect / DevOps / SRE / Performance / HFT mentor.
- Never just generate code — teach WHAT / WHY / HOW / PRODUCTION.
- Final goal: production-style platform → then HFT evolution.
- GitHub org `StockForge` with a specific repo list (created progressively).
- Both GitHub Actions and Jenkins are first-class; equivalent pipelines in both.
- Performance testing as a CI/CD quality gate (your real-work skill).
- PostgreSQL, Redis, Kafka each taught with intentional, measured failures.
- Incident engineering: we deliberately break things and follow
  Symptoms → Detection → Investigation → Root cause → Mitigation → Recovery →
  Permanent fix → Prevention.
- Two-device safety: `git pull` before work; never destructive commands; on conflicts,
  stop and explain.
- Mandatory end-of-day sequence: stop clean → tests → update README/context/state/
  prompts → commit → push → verify.
- Phase 0 first: produce the full architecture and **wait for approval** before any code.

---

## 3. Phase 0 — The architecture (written, then approved by you)

`PROJECT_CONTEXT.md` now holds the approved architecture. The headline decisions:

### Technology stack (ADR 0002 — accepted)
- **Backend:** Java 21 + Spring Boot 3.x + Maven
  (why: trading-industry standard; the JVM gives us the HFT-relevant perf curriculum —
  GC analysis, JFR, async-profiler, object pooling, lock contention)
- **Frontend:** React 18 + TypeScript + Vite
- **Realtime:** WebSocket
- **Database:** PostgreSQL 16
- **Cache:** Redis 7
- **Messaging:** Kafka 3.x (KRaft)
- **Containers:** Docker + Compose
- **Kubernetes:** kind (local) → EKS (cloud)
- **CI/CD:** GitHub Actions + Jenkins (both)
- **Perf testing:** k6 (primary), JMeter/Gatling as alternatives
- **Observability:** Prometheus + Grafana + OpenTelemetry + Loki; New Relic later
- **IaC:** Terraform + Helm
- **Perf analysis:** JFR/JMC, async-profiler, GC logs

### Repository layout (ADR 0001 — accepted)
- One service = one folder = one repo, in GitHub org `StockForge`.
- Repos created on demand: project-context (exists) → contracts → web + api → auth →
  order-service → risk-service → matching-engine → market-data → portfolio →
  notification → shared-actions → deployment → infrastructure → performance.

### The 9 application services
- **web** (UI), **api** (gateway: routing, validation, rate limiting, correlation IDs),
  **auth** (login/register/JWT/roles), **order-service** (order lifecycle),
  **risk-service** (limits, margin, position checks), **matching-engine** (order book,
  price-time priority — the performance-critical one), **market-data** (simulated
  prices + WebSocket), **portfolio** (positions, P&L, history), **notification** (WS pushes).

### CI/CD division of labor
- **GitHub Actions** = developer CI on PRs: lint → unit → integration → contract →
  security → build → container.
- **Jenkins** = enterprise side: deploy test env → E2E → performance smoke → load →
  baseline comparison → performance gate → promote.
- Performance testing lives in the delivery pipeline, exactly like your day job.

### Roadmap
- 26 phases (architecture → foundation → app → auth → orders → risk → matching →
  market data → portfolio → DB → Redis → Kafka → Docker → GitHub Actions → Jenkins →
  performance → K8s → deploy → observability → SLO → security → cloud → incidents →
  rollout → HFT evolution).

---

## 4. Day-by-day guide + GitHub setup docs

`DAY_BY_DAY_GUIDE.md` was written for **you to follow manually**. It contains:

- **Phase A — GitHub setup:** exact browser steps for creating the org and each repo,
  plus the terminal commands to push and clone.
- **Phase B — local folder structure:** the full tree of future repo folders.
- **Phase C — the day plan:** what we build each day and *why*, with production
  thinking (e.g., why contract-first, why identity is the highest-risk area, why
  missing indexes are found by measurement not guessing).
- **Production mindset cheat-sheet:** every feature has three columns —
  Local (we build) | Production (enterprises do) | Difference (what changes).

---

## 5. Where we stopped (just before creating the GitHub org)

Up to that point **nothing had been pushed to GitHub**. Everything lived only in this
local folder:

- 3 commits made locally (`b9e8cb3` Day 0, `ce26346` Day 1, `793309a` Day 1.5)
- No remote configured, no GitHub org, no GitHub repo.

The next step on Day 2 was exactly the GitHub setup in the guide.

---

## 6. What is decided vs pending

**Decided / approved (written down, versioned):**
- Name StockForge; product domain = trading; engineering-lab mentality.
- Full tech stack (Java 21 + Spring Boot + React + PostgreSQL + Redis + Kafka + Docker + kind→EKS + GitHub Actions + Jenkins + k6 + Prometheus/Grafana).
- Repo layout and two-device workflow.
- CI/CD split and performance-gate strategy.
- Phase list and 30-minute roadmap.
- Commit style note: commit messages describe the change, not the day number.

**Pending / not started (by design):**
- GitHub org/repo creation and the first push.
- All application code (starts with `stockforge-contracts`, then web + api).
- Everything after Day 3 — they unlock one day at a time.

---

## 7. Key rules we agreed to follow (recap)

1. One day = ~30 minutes. Stop clean. Record incomplete work openly.
2. Teach before automating — explain every important line.
3. Never document something as implemented if it isn't.
4. GitHub Actions AND Jenkins both first-class.
5. HFT phase: every optimization measured, never claimed without evidence.
6. Before work: pull. After work: update state files, commit, push, verify.
7. Prompts and state are project artifacts — versioned like code.
8. Do not overengineer: smallest thing that teaches the concept; split with a reason.
9. Production realism: local vs production vs difference, for every feature.
10. A new AI session reconstructs everything from these files — zero memory assumed.

---

## 8. Teach-back — Day 5: what a Spring Boot service actually is

Spring Boot is a framework that runs a normal Java web server (Tomcat) out of a tiny
program: one class annotated `@SpringBootApplication` tells it "find my beans, wire
everything, and start listening on a port". We added three moving parts to see how a
real HTTP API is shaped: a **controller** (a plain class that maps URLs to methods —
`GET /api/health` returns a JSON body), a **filter** (code that runs before and after
every request, in our case to stamp each request with a correlation ID so all its log
lines can be traced), and **structured logging** (a log format where every line carries
named fields like `correlationId=...` instead of free-form text). The reason we build
these from day one is that observability — knowing what a request did and where it
failed — cannot be bolted on later; a production gateway is judged by exactly this:
health checks for machines, correlation IDs for humans, and parseable logs for tools.
We also learned the framework's major-version move (Boot 4) reorganized test
annotations, which is why our first test compile failed and why verifying dependency
versions against Maven Central matters. (Written 2026-08-07, Day 5.)
