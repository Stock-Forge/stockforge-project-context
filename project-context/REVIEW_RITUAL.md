# STOCKFORGE — DEEP REVIEW RITUAL (one-on-one concept review)

> The project is not just about building StockForge — it is about *understanding* it.
> Every few days we stop building and **review together**, one-on-one, the way a senior
> mentor reviews with a junior engineer. The AI produces a self-contained review
> document and then we go through it together as a conversation.

---

## 1. WHEN IT HAPPENS

- **Every 5–6 completed days** (Day 0–6 → review, Day 7–12 → review, …). Adjust the
  cadence only if the user asks.
- **Whenever the user explicitly requests it** (e.g. "wrap up and explain everything so
  far", "I'm confused about X", "review the auth service with me").
- At a **natural boundary**: end of a phase, end of an incident investigation, or right
  before we start a big new technology.

## 2. WHAT IT IS

A **deep 1-on-1 teaching/mentor session**, NOT a changelog. The deliverable is a
self-contained HTML review file stored under
`stockforge-project-context/project-context/reviews/` named like
`day-0-6-concept-review.html` (range of days covered). "Self-contained" means a fresh
AI session or a browser tab can read it and understand everything — it must not depend
on conversation history.

Each review covers, for the whole range of days, with **depth over breadth**:

1. **WHAT we built** — every feature, every repo, every endpoint, in plain language.
2. **WHY** — the problem each piece solves, and *what breaks if it were gone*.
3. **HOW it works** — the actual code walked through line by line (the important lines).
4. **Concepts in depth** — the underlying engineering topic (HTTP status semantics,
   password hashing, JWTs, statelessness, Spring Security filter chains, error masking,
   PowerShell→curl quoting, etc.), explained from first principles.
5. **Diagrams** — simple SVG/inline diagrams for architecture, request flows, security
   boundaries. A picture closes the loop faster than prose.
6. **Issues & lessons** — each real incident we hit, as a mini case study: symptom →
   detection → root cause → fix → prevention → production relevance.
7. **Production mapping** — what a real Zerodha/Groww/exchange does for the same thing,
   and where our local build is similar or different (Master Prompt §41).
8. **Teach-back / homework** — 2–5 "explain it to me" questions the user must answer
   in their own words before the next day of building. The day is only "understood"
   when the user can explain it simply.

## 3. HOW THE SESSION RUNS (the ritual, step by step)

1. **AI produces the review file** in `project-context/reviews/`, then **briefs the
   user**: one paragraph per covered day, plain language, no jargon without explanation.
2. **AI walks through the review with the user interactively**, one section at a time.
   This is a *conversation*: the AI explains, then asks the user to explain back. Where
   the user hesitates or is wrong, the AI re-teaches, gently, from first principles.
3. **The user asks questions.** Nothing is too basic. If a question reveals a gap in the
   review document, the AI updates the document to fill the gap (the review file is a
   living artifact, version-controlled like everything else).
4. **Homework closes the session:** the AI gives the teach-back questions, the user
   answers them (orally or written), and the AI checks the answers. Only then is the
   review considered *passed* and building resumes.
5. **End-of-review closeout (same rules as end-of-day):** update `CURRENT_STATE.md`
   (add the review to the state), add a "Review Session" entry to `SESSION_PROMPTS.md`,
   add a line to `CHANGELOG.md`, then commit and push. A review is not done until it is
   pushed and verified.

## 4. QUALITY BAR (non-negotiable)

- **No hand-waving.** If the AI cannot explain a line of code, it must not ship that
  review section — it goes back and studies first.
- **Plain language first, code second.** Concepts before syntax.
- **Production relevance on every topic.** Every feature must answer "how do real
  platforms do this?"
- **Self-contained.** A brand-new AI session must be able to use the review as a study
  guide with zero conversation history.
- **Honest scope.** Never claim we implemented something we only described. The review
  distinguishes *what runs today* from *what is planned* (same rule as
  PROJECT_CONTEXT.md statuses).

## 5. WHERE THIS IS WIRED IN

- **`START_OF_DAY.md` Phase 3** — the end-of-day sequence references this ritual and
  reminds the AI to trigger a review when the day count hits a multiple of 5–6.
- **`MASTER_PROMPT.md`** — the master prompt references this file so every fresh session
  knows the ritual exists.

---

> Rule of thumb for the whole project: **we build so we can understand, and we review
> so the understanding sticks.** Every review is a checkpoint: it proves what we truly
> know and surfaces what we only think we know.
