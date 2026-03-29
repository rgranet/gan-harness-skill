# Design Loop Reference

Iterative Generator → Evaluator cycles to improve frontend quality.
No planner needed. Start fast, iterate hard.

---

## Setup

Before the first generation, write `harness/design-brief.md` with:
- The UI prompt (what is being built or improved)
- Target aesthetic / mood (from user or from `CLAUDE.md` design system)
- The 4 grading criteria below (copy verbatim — the language shapes output)
- Reference to the project's actual component structure and styling approach

**Important**: Read `CLAUDE.md` first. Use the project's real stack — if the project
uses React/Next.js components with Tailwind, work on actual components, not standalone
HTML files.

---

## The 4 Grading Criteria

Given to **both** Generator and Evaluator. The language steers generation.

### 1. Design Quality (HIGH weight)
Does the design feel like a coherent whole rather than a collection of parts?
Colors, typography, layout, and imagery combine to create a distinct mood and identity.
The best designs feel museum-quality — deliberate and irreducible.

### 2. Originality (HIGH weight)
Evidence of custom decisions, or template defaults left untouched?
A human designer should recognize deliberate creative choices.
**Penalize**: purple gradients over white cards, Inter+Tailwind defaults untouched,
rounded-2xl cards in a grid, generic hero with centered text+CTA. These are "AI slop."

### 3. Craft (MEDIUM weight)
Technical execution: typography hierarchy, spacing consistency, color harmony,
contrast ratios. Competence check, not creativity check. Failing = broken fundamentals.

### 4. Functionality (MEDIUM weight)
Usability independent of aesthetics. Can users understand what the interface does,
find primary actions, complete tasks without guessing?

---

## Generator Rules

```
You are the Generator. Create a distinctive, high-quality UI.

Context:
- Read harness/design-brief.md for prompt and criteria
- Read CLAUDE.md for the project's design system and stack
- [If iterating]: Read harness/eval-scores.md for latest feedback

Rules:
- Work on REAL project components — edit actual source files, not standalone HTML
- Weight Design Quality and Originality above Craft and Functionality
- Actively avoid generic AI patterns (see Originality criterion)
- After each critique, make a strategic decision:
    → REFINE if scores trend upward (push further)
    → PIVOT if the approach isn't working (try a different aesthetic)
- Write a 3-sentence rationale to harness/eval-scores.md after each iteration
- Run `npm run build` after changes to verify nothing breaks
```

---

## Evaluator Rules

```
You are the Evaluator. Grade critically.
You are skeptical. You are not generous. Grade against criteria, not vibes.

Context:
- Read harness/design-brief.md for criteria
- Read the actual component source code

Verification:
1. `npm run build` — must pass
2. If Playwright MCP available: screenshot each page/component, navigate and interact
3. Read the component markup + Tailwind classes
4. Check colors/fonts against CLAUDE.md design system
5. Verify responsive classes exist (mobile-first)
6. Check accessibility basics (contrast, semantic HTML, aria labels)

For each criterion, output:
  Score: X/10
  Reasoning: [2–4 sentences, specific, pointing to exact elements/classes]
  What must change: [concrete, actionable]

Overall: PASS (avg ≥ 7.0) or FAIL (avg < 7.0)

Append full assessment to harness/eval-scores.md.
```

---

## Iteration Loop

```
Iteration 1:
  Generator reads design-brief.md → edits real components
  Evaluator reads components → appends scores to eval-scores.md

Iteration N:
  Generator reads eval-scores.md → refines or pivots
  Evaluator grades → appends scores

Stop when:
  - Average ≥ 8.0 for 2 consecutive rounds, OR
  - Hit iteration cap (default: 10)
```

Use TodoWrite to track iteration count and scores.

---

## Calibration Tips

- **Few-shot calibration**: If evaluator is too lenient early on, add 1–2 examples:
  "This would score 4/10 on Originality because it uses default Tailwind card layout
  with no custom decisions."
- **Wording matters**: "museum-quality" in Design Quality reliably steers toward ambition.
- **Pivots are healthy**: A sharp aesthetic change mid-loop means the loop is working.
  Keep pivoted versions (git commit each iteration) — the user may prefer an earlier one.
- **If the designlint skill is available**: Run it as an additional evaluator signal.

---

## Output

At the end of the loop, present:
1. The highest-scoring version
2. Notable pivot versions the user might prefer
3. Final eval-scores.md summary
4. `git log --oneline` showing each iteration commit

Ask: "Veux-tu que je continue à itérer, ou on part sur cette version ?"
