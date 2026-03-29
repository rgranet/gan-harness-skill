---
name: gan-harness
description: >
  Multi-agent GAN-inspired harness for building and improving apps in Claude Code.
  Use when: building a full-stack app autonomously, improving UI quality through
  iterative evaluation, running a generator+critic loop for design or code,
  producing production-grade output from a short prompt.
  Trigger on: "build me an app", "create a full-stack", "improve this UI",
  "make it more polished", "autonomous build", "GAN harness", "generator evaluator",
  "sprint-based coding", "QA my app", or any non-trivial build-from-scratch request.
compatibility: claude-code
---

# GAN Harness — Multi-Agent Build & Design Loop

A structured harness for building complete, high-quality apps from a short prompt.
Inspired by Generative Adversarial Networks: a **Generator** agent builds, an
**Evaluator** agent critiques, and iteration drives quality upward.

## Modes

| User intent | Mode | Reference |
|---|---|---|
| "Make this UI better / more polished" | **DESIGN** | `references/design-loop.md` |
| "Improve this component" | **DESIGN** | `references/design-loop.md` |
| "Build me an app / tool / SaaS" | **BUILD** | `references/build-loop.md` |
| "Create a full-stack project from scratch" | **BUILD** | `references/build-loop.md` |
| "Autonomous build session" | **BUILD** | `references/build-loop.md` |

When ambiguous, ask: **"Do you want me to iterate on existing UI, or build a complete app from scratch?"**

Read the mode-specific reference before starting.

---

## Core Principles

### 1. Separate generator from evaluator
Never self-grade. Self-evaluation is systematically overconfident. Evaluation must
use a distinct adversarial prompt with explicit criteria.

### 2. Make quality gradable
"Is this good?" is unanswerable. "Does this score ≥7/10 on Originality per these
criteria?" is actionable. Anchor to explicit, enumerable rubrics.

### 3. Decompose before building
A one-line prompt → a full spec first. Never code without a plan.

### 4. Iterate, don't perfect in one pass
Even one evaluator cycle beats a single long generation. Budget 2–3 rounds minimum.

### 5. Adapt to the project
Read `CLAUDE.md` before starting. Use the project's actual stack, conventions, and
structure — never impose defaults. If the project uses Next.js, don't scaffold Vite.

### 6. Use real verification
QA means: `npm run build`, `npm run lint`, `curl` for API routes, checking types.
If Playwright MCP is available, use it for visual testing (screenshots, click-through).
If not, grade visual design by reading CSS/markup against the spec.

### 7. Watch for context anxiety
On long builds, the model may try to prematurely wrap up or cut scope. If you notice
yourself concluding early: stop, re-read spec.md, and check how many features remain.
With Opus 4.6, prefer one continuous session with automatic compaction over context resets.

### 8. Reassess harness weight
Every component encodes an assumption about what the model can't do alone. After a
model upgrade, test whether sprints/contracts are still load-bearing — simpler harness
may produce equivalent results. Remove components that no longer add value.

---

## File Communication Protocol

Agents communicate via files — no Agent SDK required:

```
harness/
├── spec.md            ← Planner writes; Generator reads
├── sprint-N.md        ← Sprint contract (Generator proposes, Evaluator reviews)
├── qa-report-N.md     ← Evaluator findings; Generator reads and fixes
├── design-brief.md    ← Design criteria + visual direction
├── eval-scores.md     ← Running scores across iterations
└── build-status.md    ← Rolling summary (for long builds, replaces old QA reports)
```

Rules:
- Write to file before handing off. Read before acting.
- Use TodoWrite to track sprint/iteration progress.
- After the build completes, offer to clean up `harness/` or keep it for reference.

---

## Cost & Time Expectations

| Mode | Typical duration | Estimated cost |
|---|---|---|
| DESIGN (5–15 iterations) | 1–4 hours | $30–$80 |
| BUILD (small app, 2–3 sprints) | 2–4 hours | $50–$130 |
| BUILD (full app, 5+ sprints) | 4–8 hours | $120–$250 |

These are rough estimates. Costs depend on model, context length, and iteration count.

---

## Quick Start

### DESIGN MODE
```
/gan-harness design

[YOUR UI PROMPT HERE]

Run at least 5 iterations.
```

### BUILD MODE
```
/gan-harness build

[YOUR APP PROMPT IN 1–4 SENTENCES]

Start with the Planner. Write spec to harness/spec.md before any code.
```
