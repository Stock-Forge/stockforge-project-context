# ADR 0001 — GitHub Organization, One-Folder-One-Repo Layout, Two-Device Workflow

**Status:** PROPOSED (Phase 0, awaiting approval)

## Context

StockForge will span multiple services, libraries, and infrastructure assets. Work happens across two devices with independent AI sessions that share no conversation history. We need a layout where each component can be built, tested, versioned, and deployed independently, and where project continuity is fully reproducible from GitHub alone.

## Decision

1. Create a GitHub Organization named `StockForge` (fallback if taken: `StockForge-Trading`, confirmed before use).
2. Each component is a **separate repository** in the org, and a **separate local folder with its own `.git`** on both devices (never a nested repo).
   - Repos are created progressively, only when their phase requires them.
3. `stockforge-project-context` is the **source of truth for continuity** (context, state, prompts, ADRs).
4. GitHub is the shared source of truth for all state; AI sessions must never assume conversation history.
5. Device workflow:
   - Before work: `git pull`, then inspect `git status` / `git branch` / `git log -5`.
   - End of every day/phase: stop clean → tests → update README → update PROJECT_CONTEXT.md (if architecture changed) → update CURRENT_STATE.md → update SESSION_PROMPTS.md → `git status` → `git diff` review → `git add` → `git commit` → `git push` → verify push.
   - Conflicts/unexpected changes: STOP and explain; no destructive Git commands.

## Alternatives

- Monorepo: simpler cross-repo refactors, but loses independent ownership/CI/deploys and the teaching value of multi-repo discipline.
- Git submodules: adds complexity without benefit here.
- Single-device only: contradicts the two-device requirement.

## Consequences

- Positive: independent CI per service; realistic enterprise layout; continuity survives device switching and even AI/provider changes.
- Negative: more Git discipline required; cross-repo changes need coordinated PRs (mitigated by `stockforge-contracts`).
