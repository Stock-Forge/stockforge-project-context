# SESSION_PROMPTS.md

> Reusable prompts for future sessions. After every session, update this file with
> a new continuation prompt and keep the **NEW SESSION RESUME PROMPT** up to date.
> These prompts are version-controlled project artifacts.

---

## NEW SESSION RESUME PROMPT (copy-paste this into a completely new AI session)

> **Every day:** use `project-context/START_OF_DAY.md` — it is the canonical daily-start
> prompt (reconstruct → work one 30-minute day → mandatory end-of-day sequence).
> The prompt below is the same protocol in compact form.

```
STOCKFORGE — NEW SESSION RESUME PROMPT

You are starting a completely new session for the StockForge project.
You have ZERO conversation history. Do NOT assume any previous context.

Step 1 — Sync:
Run: git pull
Then inspect: git status, git branch, git log -5

Step 2 — Read project context:
Read: stockforge-project-context/project-context/PROJECT_CONTEXT.md

Step 3 — Read current state:
Read: stockforge-project-context/project-context/CURRENT_STATE.md

Step 4 — Read the latest continuation prompt:
Read: stockforge-project-context/project-context/SESSION_PROMPTS.md
(Use the most recent SESSION CONTINUATION PROMPT below; ignore superseded ones.)

Step 5 — Inspect the repository currently being worked on (per CURRENT_STATE.md).

Step 6 — Reconcile: compare saved state + prompt + actual repository state.
If they disagree, investigate BEFORE writing any code.

Step 7 — Report this exact block, then continue only with the next 30-minute task:

Project reconstructed.

Current phase:
Current day:
Current repository:
Current branch:
Last completed task:
Current incomplete task:
What I understand:
Next 30-minute task:

Rules for this session:
- Teach before automating; explain WHAT/WHY/HOW/PRODUCTION.
- One day = ~30 minutes. Prefer clean stopping points. Record incomplete work openly.
- Both GitHub Actions AND Jenkins are first-class CI/CD. Do not treat Jenkins as an afterthought.
- The project is production-style AND evolves toward HFT later. Never claim the initial
  microservice architecture is exchange-grade HFT.
- End of session: stop clean → run tests → update README → update PROJECT_CONTEXT.md
  (if architecture changed) → update CURRENT_STATE.md → update SESSION_PROMPTS.md →
  git status → git diff review → git add → git commit → git push → verify push.
- Never claim the session is safely saved until git push is verified.
```

---

## SESSION CONTINUATION PROMPT TEMPLATE

After every session, append a filled copy of this template to `SESSION_PROMPTS.md`
(keep the most recent one at the top of the "Session history" section).

```
### Session N — <short summary> — <date>

Project:        StockForge
Current phase:  <phase>
Current day:    <Day N>
Current repo:   <repository>
Current branch: <branch>
Previous commit:<hash>

What was implemented:
- ...

What was learned:
- ...

Current problem / open questions:
- ...

Incomplete work (record exactly; next session continues from here):
- ...

Exact next task:
- ...

Commands to run:
- ...

Files to inspect:
- ...

Expected result:
- ...

Long-term direction:
- ...

Git verification: committed? (yes/no)  pushed? (yes/no)  verified? (yes/no)
```

---

## SESSION FORMAT (for each working day)

**Day N — Topic**

Goal: one clear 30-minute objective.

```
0–5 min     Concept
5–10 min    Production explanation
10–25 min   Implementation/testing
25–30 min   Review/documentation
```

End-of-session report:

- Completed today
- What I learned
- Production lesson
- Problems encountered
- Incomplete work
- Next 30-minute task
- Git status / commit / push status

---

## Day 0 — Foundation Session (2026-08-06)

What was implemented:

- `MASTER_PROMPT.md` — the complete StockForge scenario prompt (all 47 rules: learning approach, GitHub org, repo responsibilities, two CI/CD systems, HFT evolution, two-device git workflow, mandatory end-of-day sequence, new-session startup protocol).
- `PROJECT_CONTEXT.md` — permanent architecture/context skeleton (all `PLANNED`).
- `CURRENT_STATE.md` — Day 0 state.
- This file + `adr/` folder.

What was learned:

- Project continuity lives in GitHub, not AI memory.
- First session produces architecture only, and waits for approval.

Exact next task:

- Create GitHub organization `StockForge` + repo `stockforge-project-context`, add remote, push.
- Produce Phase 0 architecture deliverable (Master Prompt §46) into `PROJECT_CONTEXT.md`, then **WAIT FOR APPROVAL**.

Git verification: committed (yes)  pushed (no — no remote yet)  verified (no)

---

## Day 1 — Phase 0 Architecture Proposal (2026-08-06)

What was implemented:

- Wrote the full **Phase 0 architecture proposal** into `PROJECT_CONTEXT.md` (all sections marked `PROPOSED`): target architecture, repo layout (**one service = one folder = one repo**), proposed tech stack, service boundaries, API catalogue + critical traversal, event catalogue, data layer, CI/CD division (GitHub Actions + Jenkins), performance strategy, observability/SLO, security, infrastructure, incident engineering, HFT evolution, phases + repo creation order, 30-day roadmap, risks, ADR index.
- Created `adr/0001-github-org-and-repo-layout.md` and `adr/0002-technology-stack.md`.
- Created `project-context/DAY_BY_DAY_GUIDE.md` — manual GitHub setup guide + folder structure + day-by-day plan with production thinking.

What was learned:

- Architecture must be approved before implementation (Master Prompt §46).
- Backend language is the key approval point (Java 21 vs Go).

Exact next task:

- **Get user approval** of the Phase 0 proposal (especially backend language).
- Set up GitHub org + remote + push (`git remote add origin … ; git push -u origin main`).
- Clone on device B; practice the two-device workflow.
- Then begin Phase 1 repo foundation.

Result: **architecture APPROVED 2026-08-06** (Java 21 + Spring Boot confirmed). Next session = **Day 2** (GitHub org + repo foundation) per `DAY_BY_DAY_GUIDE.md`.

Git verification: committed (yes)  pushed (no — no remote yet)  verified (no)

---

## Day 1.5 — Architecture Approval + Day-by-Day Guide (2026-08-06)

What was implemented:

- Phase 0 architecture **APPROVED** by the user (stack as proposed: Java 21 + Spring Boot, React+TS, PostgreSQL, Redis, Kafka, Docker, kind→EKS, GitHub Actions + Jenkins, k6, Prometheus/Grafana).
- ADR 0001 + 0002 status → ACCEPTED.
- Created `DAY_BY_DAY_GUIDE.md` (manual: GitHub org/repo setup, folder structure, day plan with production thinking).
- Updated `PROJECT_CONTEXT.md` statuses to APPROVED and `CURRENT_STATE.md` to reflect approval.

What was learned:

- The user does GitHub/repo creation manually; the AI builds locally and teaches.

Exact next task (Day 2):

- User creates GitHub org `StockForge` + repo `stockforge-project-context` (empty), runs:
  ```
  git remote add origin https://github.com/Stock-Forge/stockforge-project-context.git
  git push -u origin main
  ```
- Clone on Device B; practice two-device git round-trip.
- Then Phase 1 (stockforge-contracts, Day 3).

Git verification: committed (yes)  pushed (no — no remote yet)  verified (no)

---

## Day 2 — GitHub org + repo foundation + git workflow (2026-08-06)

What was implemented:

- GitHub org **`Stock-Forge`** created (hyphenated; product name stays `StockForge`).
- Repo `stockforge-project-context` created (empty), remote `origin` added, `main` pushed + tracking set (`git push -u origin main`).
- Docs updated to `Stock-Forge` URLs.
- 30-minute roadmap expanded to Day 39+ (PROJECT_CONTEXT.md §21).
- Git workflow practiced: branch → commit → push -u → merge (fast-forward) → push → delete local+remote branch.
- `CHANGELOG.md` created with Day 0-2 entries.
- Commit `89d4b4f` on `origin/main`.

What was learned:

- Push output on PowerShell shows as "git :" red text — that is stderr formatting, not an error; verify with `git status -sb` and `git ls-remote --heads origin`.
- Feature branches merge cleanly when main hasn't moved (fast-forward).

Exact next task (Day 3):

- **User:** create empty repo `stockforge-contracts` in `Stock-Forge`.
- **AI:** create local `stockforge-contracts` (own repo), OpenAPI spec + 7 event schemas + README, push; update state + CHANGELOG here, push.

Git verification: committed (yes)  pushed (yes)  verified (yes)

---

## Day 3 — Upcoming: `stockforge-contracts` (next session pointer)

Current state at the end of Day 2 / before Day 3 (see `CURRENT_STATE.md`):

- Repo `stockforge-project-context` on `origin/main` at `cb9af31`, clean tree, 9 commits.
- Central-state rule enforced: all state updates happen in this repo, never in the
  working repo; both repos are committed + pushed each day.
- Next: Day 3 — create repo `stockforge-contracts` (user creates empty repo on GitHub),
  AI builds `contracts/openapi.yaml` + `contracts/events/` (7 events) + README in a new
  local repo, pushes it, then updates THIS repo's state to point at Day 4.
- Begin with the full `START_OF_DAY.md` paste.

Git verification: not applicable (pointer entry — no code changed).
