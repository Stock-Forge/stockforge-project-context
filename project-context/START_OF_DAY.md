# STOCKFORGE — START OF DAY PROMPT

> Copy-paste this file's content into a completely new AI session **at the start of
> every working day**. It makes the AI follow the startup protocol (Master Prompt §31),
> reconstruct the project from files alone, then work a single ~30-minute day and
> finish with the mandatory end-of-day sequence (§29, §47).
>
> Do not edit the instructions below every day. This file is permanent and generic;
> the AI pulls the day-specific facts from `CURRENT_STATE.md` and `SESSION_PROMPTS.md`.

---

STOCKFORGE — START OF DAY PROMPT

You are starting a new working session for the StockForge project.
You have ZERO conversation history. Do NOT assume any previous context.

## PHASE 1 — RECONSTRUCT (before any work)

Run or inspect, in this order:

1. `git pull` — sync with the GitHub source of truth. If it fails or shows conflicts,
   STOP and report; never use destructive Git commands.
2. Read `stockforge-project-context/project-context/PROJECT_CONTEXT.md` — the permanent
   architecture/context document.
3. Read `stockforge-project-context/project-context/CURRENT_STATE.md` — the live progress
   state (this tells you the current phase, day, and exact next task).
4. Read the latest continuation prompt in
   `stockforge-project-context/project-context/SESSION_PROMPTS.md` (the most recent
   "Session N" entry; ignore superseded ones).
5. Read `stockforge-project-context/project-context/ISSUES_LOG.md` — know the pitfalls
   and fixes from previous days so they are not repeated.
6. If working on another repo (e.g. `stockforge-contracts`), read its README and inspect
   its structure too.
7. Check `git status`, `git branch`, `git log -5` in every repo you will touch.
8. Reconcile: compare the saved state + prompts against the actual repository state.
   If they disagree, investigate and report BEFORE writing any code.

Then report this exact block:

```
Project reconstructed.

Current phase:
Current day:
Current repository:
Current branch:
Last completed task:
Current incomplete task:
What I understand:
Next 30-minute task:
```

## PHASE 2 — WORK ONE DAY (~30 MINUTES)

- **Begin by briefing the user in FULL detail (plain language, BEFORE any code or
  commands):**
  - **WHAT** we are doing today and **HOW it helps the project** — where today's work
    sits in the roadmap, why it exists, and what it unlocks next.
  - **Every step** we will take today, in order, with the exact commands and files.
  - For each step: **which technology from our stack we are using** (Vite + React + TS,
    Spring Boot, PostgreSQL, Redis, Kafka, Docker, GitHub Actions/Jenkins, k6, …) and
    why that tool exists.
  - **How this compares to a production-grade trading system like Zerodha or Groww** —
    what those platforms do for this same thing, and how our local build is similar to
    or different from theirs.
- Follow the day's plan from `DAY_BY_DAY_GUIDE.md` (the section for the current day).
- **State a hypothesis before EVERY change:** "I believe X happens because of Y" → do the
  change → measure/observe → confirm or revise. No guessing. (Applies to bugs, config,
  perf, and feature work alike — PROJECT_CONTEXT §25 habit 2.)
- **Go deep, not wide:** for today's topic ask "what problem does this solve and what
  breaks if it's gone?" (Kafka → partitioning/ordering/exactly-once; JVM → memory
  model/GC/JIT; order book → price-time priority — §25 habit 3).
- **Weekly habits (pick one if this is that day):** real-platform reading — study how
  Zerodha/Groww/Coinbase or an exchange does today's domain and compare to ours (§25
  habit 6); primitive side quest — build one toy version of a tool we use (§26).
- Goal: ONE clear 30-minute objective. Do not exceed it. Prefer a clean stopping point.
- Teach while building (Master Prompt §3): WHAT we build → WHY → HOW → how production
  teams do it → what can go wrong → how we detect/fix it. Explain every important line
  of generated artifacts (Dockerfile, Kubernetes YAML, Jenkinsfile, Actions YAML, etc.).
- If the day's task cannot finish: stop at a clean point and record the incomplete work
  exactly (reason, current state, what remains, next action). Do NOT rush to "complete"
  at the cost of the 30-minute limit.
- If something unexpected is found in the repo (uncommitted changes, diverged branch,
  failing state): STOP and explain; do not overwrite or force anything.

## PHASE 3 — END OF DAY (mandatory sequence, no exceptions)

**CENTRAL-STATE RULE: ALL project state lives in `stockforge-project-context`.**
Even when today's work is done in a different repo (e.g. `stockforge-contracts`),
steps 3-7 below are performed HERE, in `stockforge-project-context`, and pushed HERE.
The working repo only receives its own code, tests, and its own README — it never
receives project state files. Both repos are committed and pushed each day.

1. Stop at a clean point.
2. Run the appropriate tests (in whichever repo the work was done).
3. Update the working repo's README if functionality changed (own README only).
4. **Here in `stockforge-project-context`:** update `PROJECT_CONTEXT.md` if architecture
   changed (mark statuses correctly).
5. **Here:** update `CURRENT_STATE.md` (date, phase, day, repo worked on, branch, commit,
   completed, incomplete, files changed, tests, next exact task).
6. **Here:** add a "Session N" entry to `SESSION_PROMPTS.md` and update the Day N entry
   in `DAY_BY_DAY_GUIDE.md` (mark it done, note what remains).
7. **Here:** update `ISSUES_LOG.md` — record every issue we faced today as a mini
   case study: symptom, detection, cause, fix, prevention, production relevance (what a
   real Zerodha/Groww-style platform would do). One entry per issue; "none" only if
   truly none.
8. **Here:** teach-back — write today's concept in your own words (2-5 sentences) into
   `JOURNEY_SO_FAR.md`, as if explaining to a smart friend. If you can't write it
   simply, the day is not complete (§25 habit 1).
9. **Here:** update `CHANGELOG.md` with one line for the day.
10. `git status` — review what changed (in this repo AND the working repo).
11. `git diff` — review the changes themselves.
12. `git add`
13. `git commit` — **commit message describes the change, NOT the day number**
    (e.g. "Add OpenAPI spec for orders API", not "Day 3").
14. `git push` — push THIS repo; push the working repo too.
15. Verify both pushes: `git status -sb` shows up to date; confirm on GitHub if possible.

Before you declare the session complete, verify the working tree is clean and the
latest commit is on the remote. If something is intentionally uncommitted, say so
explicitly. Never claim the session is safely saved until the push is verified.

## LONG-TERM RULES TO OBEY

- Both GitHub Actions AND Jenkins are first-class CI/CD — never treat Jenkins as an
  afterthought.
- The project is production-style AND evolves toward HFT later. Never claim the initial
  microservice architecture is exchange-grade HFT.
- One service = one folder = one repo. Create repos only when their phase needs them.
- Do not overengineer; smallest architecture that teaches the concept.
- Every performance optimization: baseline → hypothesis → change → measurement →
  result → conclusion.
- Never document something as implemented when it is not.
- The next session depends on your Phase 3 updates — do them properly.
