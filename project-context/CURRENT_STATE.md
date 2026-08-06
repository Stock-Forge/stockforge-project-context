# CURRENT_STATE.md

> Current progress state. Updated after every meaningful session.
> Read this first (after PROJECT_CONTEXT.md) when starting a new AI session.

---

## Snapshot

| Field | Value |
|---|---|
| Date | 2026-08-06 |
| Phase | Phase 0 — Architecture and planning |
| Day | Day 1 |
| Current repository | stockforge-project-context |
| Current branch | main |
| Current commit | See `git log -1` / `git rev-parse HEAD` |

---

## What Day 1 completed

- **Phase 0 architecture proposal** written into `project-context/PROJECT_CONTEXT.md` (all proposal sections marked `PROPOSED`):
  - Target system + CI/CD architecture (both GitHub Actions and Jenkins first-class)
  - Repositories + local layout rule: **ONE SERVICE = ONE FOLDER = ONE REPO** (separate git repo per component, created on demand)
  - Proposed technology stack (Java 21 + Spring Boot, React + TS + Vite, PostgreSQL, Redis, Kafka, Docker/Compose, kind→EKS, k6, Prometheus/Grafana/OTel, Terraform/Helm)
  - Service boundaries with split rationale
  - API catalogue + `POST /api/orders` critical traversal
  - Event catalogue (producers/consumers)
  - Data layer, Docker/K8s, CI/CD division, performance strategy, observability/SLO, security, infrastructure, incidents, HFT evolution, phases, repo creation order, 30-day roadmap, risks, ADR index
- Created ADRs:
  - `adr/0001-github-org-and-repo-layout.md` — org + repo-per-service + two-device workflow (PROPOSED)
  - `adr/0002-technology-stack.md` — full stack (PROPOSED)

## What is deliberately NOT done

- No application code (per Master Prompt §46 — wait for approval).
- No GitHub org/repositories created yet (user is creating them manually).
- No architecture approved yet.

## Incomplete work

```
INCOMPLETE
Reason: Master Prompt §46 — architecture must be approved before implementation.
Current state: Phase 0 proposal written and committed.
What remains:
  - User approval of PROJECT_CONTEXT.md (especially backend language: Java vs Go)
  - Create GitHub org "StockForge" + repo stockforge-project-context, add remote, push
  - Decide GitHub org name fallback if "StockForge" is taken
Next action: See "Next 30-minute task" below.
```

## Git status

Working tree should be clean after Day 1 commit. Verify on the other device:

```
git pull
git status
git branch
git log -5
```

## Next 30-minute task (Day 2)

**Pending approval first.** After the user approves the Phase 0 architecture:

1. (If not done) Create GitHub org `StockForge` + repo `stockforge-project-context` (empty, no README).
2. Add remote + push:
   ```
   git remote add origin https://github.com/StockForge/stockforge-project-context.git
   git push -u origin main
   ```
3. On device B: `git clone https://github.com/StockForge/stockforge-project-context.git`.
4. Practice the two-device workflow (pull → work → commit → push → verify).
5. Update the 30-minute roadmap (PROJECT_CONTEXT.md §21) based on approval feedback.

## Next 30-minute plan

1. Get approval on architecture (user reviews PROJECT_CONTEXT.md + ADRs).
2. Set up GitHub org + remote + push (steps above).
3. Begin Phase 1 repo foundation if time remains.
