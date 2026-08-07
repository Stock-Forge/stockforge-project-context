# StockForge — Changelog

All notable changes per project day. Format: date — Day N — summary.

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
