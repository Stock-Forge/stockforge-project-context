# stockforge-project-context

**StockForge** — production-grade stock trading platform and engineering laboratory.

This repository is the **source of truth for project continuity**. Session state,
architecture context, and AI prompts live here so that any AI session, on any
device, can reconstruct the entire project with zero conversation history.

## Repository layout

```
stockforge-project-context/
│
├── README.md
└── project-context/
    ├── MASTER_PROMPT.md      # The complete StockForge master prompt
    ├── PROJECT_CONTEXT.md    # Permanent architecture/context document
    ├── CURRENT_STATE.md      # Current progress state (update after every session)
    ├── SESSION_PROMPTS.md    # Resume prompt + continuation-prompt history
    └── adr/                  # Architecture Decision Records
```

## How to start a new AI session

Follow the **NEW SESSION RESUME PROMPT** in `project-context/SESSION_PROMPTS.md`:

1. `git pull`
2. Read `PROJECT_CONTEXT.md`
3. Read `CURRENT_STATE.md`
4. Read latest continuation prompt in `SESSION_PROMPTS.md`
5. Inspect `git status` / `git branch` / `git log -5`
6. Reconcile state vs prompt vs repo
7. Continue with the next 30-minute task

## Two-device workflow

- GitHub is the shared source of truth.
- Before working: `git pull`.
- After every day/phase: update state files → commit → push → verify.

## Status

Phase 0 — Architecture and planning (foundation). See `project-context/CURRENT_STATE.md`.

## Known limitations

- No GitHub org/repo created yet — remote not configured, nothing pushed.
- No application code (by design — Phase 0 is architecture only).
