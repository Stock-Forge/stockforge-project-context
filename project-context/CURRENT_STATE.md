# CURRENT_STATE.md

> Current progress state. Updated after every meaningful session.
> Read this first (after PROJECT_CONTEXT.md) when starting a new AI session.

---

## Snapshot

| Field | Value |
|---|---|
| Date | 2026-08-06 |
| Phase | Phase 1 — Git/GitHub organization and repository foundation |
| Day | Day 2 (complete) |
| Current repository | stockforge-project-context |
| Current branch | main |
| Current commit | `89d4b4f` (pushed to `origin/main`) |

---

## What Day 2 completed

- **GitHub org created:** `Stock-Forge` (https://github.com/Stock-Forge) — hyphenated; project/product name stays `StockForge`.
- **Repo created + pushed:** `stockforge-project-context` (empty on GitHub, no README), remote `origin` added, `main` pushed and tracked (`git remote -v` shows origin; `git push -u origin main` done).
- Docs updated to actual org name `Stock-Forge` (PROJECT_CONTEXT.md, DAY_BY_DAY_GUIDE.md, ADR 0001, CURRENT_STATE, SESSION_PROMPTS).
- **30-minute roadmap expanded** to Day 39+ in PROJECT_CONTEXT.md §21 (added Days 11-39: risk, matching, market-data, portfolio, notification, integration, DB/Redis/Kafka labs, Docker, GitHub Actions, Jenkins, perf gates, Kubernetes, observability, security, cloud, incidents, rollout, HFT).
- **Git workflow practiced (real):** `git switch -c feature/changelog` → add/commit → `git push -u origin <branch>` → switch to main → merge (fast-forward) → push → delete local + remote branch.
- Created `CHANGELOG.md` (project milestone log).

## What is deliberately NOT done

- **Device B clone + round-trip not yet exercised** — user needs to run on the other machine:
  ```
  cd C:\CODE
  git clone https://github.com/Stock-Forge/stockforge-project-context.git
  cd stockforge-project-context
  git log --oneline
  ```
- No application code (starts Day 3/4 with contracts + web/api).
- Branch protection + PR workflow not enabled (planned for Phase 14 / GitHub Actions).

## Incomplete work

```
INCOMPLETE
Reason: Two-device round-trip requires physical action on Device B (user-owned).
Current state: main pushed; workflow practiced on Device A.
What remains:
  - Clone on Device B and verify `git log` matches (b9e8cb3, ce26346, 793309a, 657fb05, 89d4b4f)
  - Optional: repeat a branch round-trip from Device B to prove pull/merge
Next action: See "Next 30-minute task (Day 3)" below.
```

## Git status

```
git status        # clean
git branch -a     # only main (local + origin/main)
git log --oneline # b9e8cb3 → ce26346 → 793309a → 657fb05 → 89d4b4f
```

## Next 30-minute task (Day 3)

**stockforge-contracts — API & event contracts (Phase 1 foundation).**

1. **You (manual):** on GitHub create empty repo `stockforge-contracts` (no README) in org `Stock-Forge`.
2. AI session:
   - startup protocol (git pull, read context/state/prompts)
   - create local folder `stockforge-contracts` (own git repo, sibling to this one)
   - `contracts/openapi.yaml` (auth + orders + portfolio endpoints) + `contracts/events/*` (7 Kafka events) + README explaining contract-first
   - commit + push to `Stock-Forge/stockforge-contracts`
   - update this repo's state/prompts + CHANGELOG, commit + push

## Next 30-minute plan

1. Device B clone verification (user).
2. Day 3: stockforge-contracts (see above).

---

## HOW TO BEGIN THE NEXT DAY (paste this into a new AI session)

**Rule for the user:** copy-paste the FULL content of
`stockforge-project-context\project-context\START_OF_DAY.md` into the new AI session.
That file IS the begin-day prompt. It does not need editing each day.

**CENTRAL-STATE RULE (also baked into START_OF_DAY.md):** project state is updated
ONLY in this repository (`stockforge-project-context`) — `CURRENT_STATE.md`,
`SESSION_PROMPTS.md`, `PROJECT_CONTEXT.md`, `DAY_BY_DAY_GUIDE.md`, `CHANGELOG.md`.
We often work in a DIFFERENT repo that day (e.g. `stockforge-contracts`); that repo
gets only its own code, tests, and README. State updates happen HERE, always, and are
pushed HERE. Both repos are committed and pushed every day.

**Quick-start summary for the new AI (START_OF_DAY.md expands this):**

```
StockForge. New session, zero memory.

1. git pull in stockforge-project-context (and in any other repo being touched).
2. Read PROJECT_CONTEXT.md, CURRENT_STATE.md, latest entry in SESSION_PROMPTS.md.
3. Check git status / branch / log -5. Reconcile state vs prompt vs repo. Investigate
   before coding if they disagree. Report: Project reconstructed + phase/day/repo/
   branch/last task/incomplete/next task.
4. Work ONE ~30-minute day per DAY_BY_DAY_GUIDE.md. Teach while building. If not
   finished, record incomplete work openly (reason/state/remains/next).
5. End of day (mandatory):
   - run tests
   - update the working repo's own README
   - CENTRAL-STATE RULE: update PROJECT_CONTEXT.md (if architecture changed),
     CURRENT_STATE.md, SESSION_PROMPTS.md, DAY_BY_DAY_GUIDE.md, CHANGELOG.md
     HERE in stockforge-project-context — never in the working repo
   - git status + git diff review
   - git add + git commit (message describes the change, not the day)
   - git push in BOTH repos; verify (git status -sb up to date)
   - never claim saved until pushes are verified
```
