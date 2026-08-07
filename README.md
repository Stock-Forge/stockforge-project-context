# stockforge-project-context

**StockForge** — production-grade stock trading platform and engineering laboratory.

This repository is the **source of truth for project continuity**. Session state,
architecture context, and AI prompts live here so that any AI session, on any
device, can reconstruct the entire project with zero conversation history.

## Repository layout

```
stockforge-project-context/
│
├── README.md                  # This file
├── CHANGELOG.md               # One line per project day
└── project-context/
    ├── START_OF_DAY.md        # COPY-PASTE THIS EVERY DAY to start a session
    ├── MASTER_PROMPT.md       # The complete StockForge master prompt
    ├── PROJECT_CONTEXT.md     # Permanent architecture/context document (APPROVED)
    ├── CURRENT_STATE.md       # Current progress state (update after every session)
    ├── SESSION_PROMPTS.md     # Resume prompt + continuation-prompt history
    ├── DAY_BY_DAY_GUIDE.md    # Manual companion: GitHub setup + day-by-day plan
    ├── ISSUES_LOG.md          # Every issue faced, day by day (symptom/cause/fix/production)
    ├── JOURNEY_SO_FAR.md      # Plain-language summary of everything done so far
    └── adr/                   # Architecture Decision Records
```

## How to start each day (both devices)

1. `git pull`
2. **Copy-paste `project-context/START_OF_DAY.md` into the new AI session.**
   It reconstructs the project from these files, works one ~30-minute day, and runs
   the mandatory end-of-day sequence (update state → issues log → tests → README →
   commit → push → verify) — all state files update automatically, no prompts needed.
3. Do any manual steps listed in the current-day section of `DAY_BY_DAY_GUIDE.md`.

## Two-device workflow

- GitHub is the shared source of truth (`https://github.com/Stock-Forge`).
- Before working: `git pull`.
- After every day/phase: update state files → commit → push → verify.
- Commit messages describe the change, not the day number.

## Status

Phase 1 — Git/GitHub organization and repository foundation. Day 4 complete
(`stockforge-web` pushed); Day 5 (`stockforge-api`) deferred to the personal PC
(JDK 21 setup). The 10 "0.1%" engineering habits are now part of the project
(`PROJECT_CONTEXT.md` §25-26).
See `project-context/CURRENT_STATE.md` for the live state.

## Known limitations

- Device B clone + round-trip not yet exercised (see Day 2 section of the guide).
- `stockforge-web` is a static shell — no API wiring yet (Day 5+).
- This device has Java 17 + no admin: JVM work happens on the personal PC until
  containers (Phase 12-13) make toolchains device-independent.
