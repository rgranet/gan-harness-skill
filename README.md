# GAN Harness

**Multi-agent Generator-Evaluator loop for building high-quality apps in Claude Code.**

By [Ruben Granet](https://github.com/rgranet) · v1.0.0 · MIT License

---

## The Problem

AI coding agents build fast but plateau early. Left unconstrained:

**Architecture:** Flat file structure · No separation of concerns · Everything in one component · Hardcoded values everywhere

**Quality:** Works on happy path · No edge cases · Display-only features · Silent bugs · "Demo-grade" depth

**Design:** Inter + Tailwind defaults · Purple gradients · `rounded-2xl` card grids · Generic hero sections · Fade-in animations

A single generation pass cannot self-correct. Self-evaluation is systematically overconfident — models praise their own output even when quality is obviously mediocre.

**GAN Harness** breaks this ceiling by separating creation from judgment. A Generator builds, an Evaluator critiques against explicit rubrics, and iteration drives quality upward — the same adversarial dynamic that powers GANs.

Based on [Anthropic's engineering blog post on harness design for long-running agentic coding](https://www.anthropic.com/engineering/harness-design-long-running-apps).

---

## Two Modes

### Build Mode
*"Build me a crypto portfolio tracker with real-time prices"*

Three-role pipeline: **Planner** expands a 1-sentence prompt into a full product spec, **Generator** implements features sprint by sprint, **Evaluator** runs QA after each sprint with concrete verification (build checks, API tests, code review). Loop until all sprints pass.

### Design Mode
*"Make this landing page feel more premium"*

Fast iteration loop: **Generator** edits real project components, **Evaluator** grades against 4 criteria (Design Quality, Originality, Craft, Functionality). Loop until avg score >= 8.0 for 2 consecutive rounds.

---

## Before / After

Without this skill, an agent asked to "build a task management SaaS" produces:

```
One pass → 45 min → compiles but:
- Auth is display-only (no real sessions)
- Database queries are stubbed
- 3 of 8 features are TODO comments
- UI is default Tailwind
- No error handling
- Agent says "Done! Everything works great."
```

With this skill, the same prompt triggers:

```
Planner   → 16-feature spec with design language, sprint groups, AI integration
Sprint 1  → Generator builds core features → Evaluator finds 4 bugs → fixes applied
Sprint 2  → Generator builds dashboard → Evaluator catches missing edge cases → fixed
Sprint 3  → Generator adds AI features → Evaluator verifies API wiring end-to-end
Final     → Clean build, typed, real data flow, tested API routes
           4 hours · $120 · production-grade
```

---

## What's Inside

```
gan-harness/
├── SKILL.md                          # Core protocol (mode selection + 8 principles)
├── README.md                         # This file
└── references/
    ├── build-loop.md                 # Full-stack: Planner → Generator → Evaluator
    └── design-loop.md                # UI quality: Generator → Evaluator iterations
```

---

## Install

### Claude Code (native)

Copy `gan-harness/` into your project's `.claude/skills/` directory:

```bash
cp -r gan-harness/ .claude/skills/gan-harness/
```

### Manual (any agent)

Reference `SKILL.md` in your agent's system prompt and include the `references/` folder.

---

## How It Works

### Build Mode — 3 roles, sprint loop

| Phase | Role | Input | Output |
|-------|------|-------|--------|
| 1 | **Planner** | 1-4 sentence prompt + `CLAUDE.md` | `harness/spec.md` (8-16 features, design language, sprints) |
| 2 | **Generator** | spec + previous QA report | Working code + `harness/sprint-N.md` (contract) |
| 3 | **Evaluator** | Code + sprint contract | `harness/qa-report-N.md` (scores, bugs, required fixes) |

Loop: Generator builds → Evaluator grades → if FAIL, Generator fixes (max 2 rounds) → if PASS, next sprint.

**QA is real, not theater.** The Evaluator runs `npm run build`, `npm run lint`, `curl` against API routes, reads code for type safety, and uses Playwright MCP for visual testing (if available).

### Design Mode — 4 criteria, iterative

| Criterion | Weight | What it catches |
|-----------|--------|-----------------|
| Design Quality | HIGH | Incoherent aesthetics, parts that don't form a whole |
| Originality | HIGH | Template defaults, AI slop patterns, generic layouts |
| Craft | MEDIUM | Broken typography hierarchy, spacing inconsistency |
| Functionality | MEDIUM | Unclear actions, unusable flows |

Generator edits real project components (not standalone HTML). Evaluator grades against criteria. Stop at avg >= 8.0 for 2 consecutive rounds or iteration cap (default: 10).

---

## Core Principles

| # | Principle | Why |
|---|-----------|-----|
| 1 | Separate generator from evaluator | Self-evaluation is overconfident |
| 2 | Make quality gradable | "Is this good?" is unanswerable; rubrics are actionable |
| 3 | Decompose before building | No code without a plan |
| 4 | Iterate, don't perfect in one pass | Even 1 eval cycle beats a single long generation |
| 5 | Adapt to the project | Read `CLAUDE.md`, use the real stack |
| 6 | Use real verification | Build checks, linting, curl, Playwright — not vibes |
| 7 | Watch for context anxiety | Detect and counter premature wrap-up on long builds |
| 8 | Reassess harness weight | Strip components that newer models handle natively |

---

## Cost & Time

| Mode | Duration | Estimated cost |
|------|----------|----------------|
| Design (5-15 iterations) | 1-4 hours | $30-$80 |
| Build (small, 2-3 sprints) | 2-4 hours | $50-$130 |
| Build (full, 5+ sprints) | 4-8 hours | $120-$250 |

---

## Designed For

- **Opus 4.6** — continuous session with automatic compaction, no manual context resets
- **Playwright MCP** — visual QA if available, graceful fallback to code review if not
- **DesignLint skill** — integrates as additional evaluator signal when installed

---

## License

MIT
