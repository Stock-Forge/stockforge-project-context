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
5. If working on another repo (e.g. `stockforge-contracts`), read its README and inspect
   its structure too.
6. Check `git status`, `git branch`, `git log -5` in every repo you will touch.
7. Reconcile: compare the saved state + prompts against the actual repository state.
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

- Follow the day's plan from `DAY_BY_DAY_GUIDE.md` (the section for the current day).
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

1. Stop at a clean point.
2. Run the appropriate tests.
3. Update the relevant repo README(s) if functionality changed.
4. Update `PROJECT_CONTEXT.md` if architecture changed (mark statuses correctly).
5. Update `CURRENT_STATE.md` (date, phase, day, repo, branch, commit, completed,
   incomplete, files changed, tests, next exact task).
6. Add a "Session N" entry to `SESSION_PROMPTS.md` and update the Day N entry in
   `DAY_BY_DAY_GUIDE.md` (mark it done, note what remains).
7. Update `CHANGELOG.md` with one line for the day.
8. `git status` — review what changed.
9. `git diff` — review the changes themselves.
10. `git add`
11. `git commit` — **commit message describes the change, NOT the day number**
    (e.g. "Add OpenAPI spec for orders API", not "Day 3").
12. `git push`
13. Verify the push: `git status -sb` shows up to date; confirm on GitHub if possible.

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
