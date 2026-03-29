# Build Loop Reference

Full-stack autonomous build: Planner → Generator (sprints) → Evaluator (QA per sprint).
Three roles, one session, file-based handoffs.

---

## The 3 Roles

You play all 3 roles sequentially. Announce each transition explicitly.

```
[PLANNER]    → writes harness/spec.md
[GENERATOR]  → builds features sprint by sprint
[EVALUATOR]  → tests the build, writes harness/qa-report-N.md
```

---

## Phase 1 — Planner

**Input**: 1–4 sentence user prompt + `CLAUDE.md` (project context)
**Output**: `harness/spec.md`

### Rules:
- Read `CLAUDE.md` first — use the project's actual stack, conventions, and structure
- Expand prompt into a full product spec with 8–16 named features
- Be ambitious — under-scoping is the default failure mode
- Focus on WHAT (user stories, product context), not HOW (implementation details)
- Organize into sprints of 3–5 features each
- If no visual direction specified, define: color palette, typography, 2–3 aesthetic adjectives
- If the designlint skill is available, reference it for design evaluation

### spec.md structure:
```markdown
# [App Name] — Product Spec

## Overview
[2–3 sentence product vision]

## Design Language
- Palette: [colors]
- Typography: [font choices + hierarchy]
- Aesthetic: [adjectives]

## Features

### Sprint 1: [Theme]
- Feature 1: [user story]
- Feature 2: [user story]
- Feature 3: [user story]

### Sprint 2: [Theme]
...

## Stack
[Read from CLAUDE.md — use the project's existing stack, not defaults]

## AI Integration
[Describe Claude-powered features if applicable]
```

---

## Phase 2 — Generator (sprint loop)

**Input**: `harness/spec.md` + previous `harness/qa-report-N.md` (if any)
**Output**: Working code + `harness/sprint-N.md`

### Before each sprint — write the contract

`harness/sprint-N.md`:
```markdown
# Sprint N Contract

## Features being built
[List from spec]

## Definition of done
[Testable behaviors bridging user story → implementation]

## Verification plan
[How the Evaluator will verify — must be things Claude Code can actually do]
Examples:
- `npm run build` passes with no errors
- `curl -X POST /api/endpoint` returns 201 with expected shape
- Component renders correct markup (read the output)
- TypeScript types are correct (`npm run lint` passes)
- Database query returns expected rows
```

### Generator rules:
- One sprint at a time — never jump ahead
- Commit at the end of each sprint: `git commit -m "Sprint N: [theme]"`
- Self-check before QA: does `npm run build` pass? Do the types check?
- Stub complex items clearly — a noted stub beats a silent bug
- Update TodoWrite after each sprint

---

## Phase 3 — Evaluator (QA per sprint)

**Input**: Code + `harness/sprint-N.md`
**Output**: `harness/qa-report-N.md`

### Verification methods:

1. **Build check**: `npm run build` — must pass clean
2. **Lint/type check**: `npm run lint` or `npx tsc --noEmit`
3. **API testing**: `curl` or `httpie` against running dev server
4. **Visual testing (if Playwright MCP available)**: Navigate pages, take screenshots, click through flows. This is the strongest QA signal — use it if available.
5. **Code review**: Read source, check for dead code, missing error handling, type issues
6. **Database**: Query Supabase/SQLite directly to verify schema and data
7. **Structure check**: Verify files exist, exports are correct, imports resolve

> **Note**: Out of the box, Claude is a poor QA agent — it tends to test superficially.
> Playwright MCP dramatically improves evaluation quality. If unavailable, compensate
> with stricter code review and explicit test criteria.

### Scoring criteria:

```
1. Feature Completeness (HIGH weight)
   All sprint contract items implemented and functional?
   Stubbed items clearly noted — silent stubs FAIL.

2. Code Quality (HIGH weight)
   Build passes. Types check. No console.error spam.
   No hardcoded secrets. Components reasonably sized.

3. Visual Design (MEDIUM weight)
   Code matches the design language in spec.md?
   Check CSS/Tailwind classes against spec palette and typography.
   Generic AI-slop patterns (see design-loop.md criteria) → FAIL.

4. Product Depth (MEDIUM weight)
   Does this feel like a real app or a demo?
   Can a user accomplish the core task this sprint enables?
```

### qa-report-N.md structure:
```markdown
# QA Report — Sprint N

## Verification Results
- `npm run build`: PASS/FAIL [output if failed]
- `npm run lint`: PASS/FAIL
- API tests: [curl results]

## Scores
| Criterion | Score | Notes |
|---|---|---|
| Feature Completeness | X/10 | |
| Code Quality | X/10 | |
| Visual Design | X/10 | |
| Product Depth | X/10 | |
| **Average** | **X/10** | |

## Result: PASS / FAIL
(PASS = all criteria ≥ 6/10 AND avg ≥ 7/10)

## Bugs found
- [FAIL] Feature: [what] → [actual behavior] → [file:line]

## Required fixes before Sprint N+1
[Prioritized list]
```

### Evaluator disposition:
- Be skeptical. Features that "look like they work" but aren't wired → FAIL.
- Test edge cases, not just happy path.
- If you catch yourself saying "minor" — write it down anyway.
- Display-only with no interactivity → NOT implemented.
- If Playwright MCP is available: take screenshots, navigate pages, click through flows.
- If not: grade visual design by reading CSS/markup against spec — acknowledge limitation.

---

## Sprint Loop

```
For each sprint in spec.md:

  [GENERATOR]
    1. Read spec.md + previous qa-report (if any)
    2. Write harness/sprint-N.md (contract)
    3. Build features
    4. Self-check: npm run build && npm run lint
    5. Commit: git commit -m "Sprint N: [theme]"

  [EVALUATOR]
    1. Read sprint-N.md
    2. Run verification suite (build, lint, curl)
    3. Read code and check against contract
    4. Write qa-report-N.md
    5. If FAIL → Generator fixes (max 2 fix rounds per sprint)
    6. If PASS → next sprint
```

---

## Context Management

**With Opus 4.6**: Prefer one continuous session with automatic compaction. No need
for manual context resets. The model plans more carefully and sustains agentic tasks longer.

**Watch for context anxiety**: On long builds, the model may try to prematurely wrap
up or cut scope ("let's finish here", "that covers the basics"). If you notice this:
re-read spec.md, check remaining features, and continue.

For long builds (6+ sprints):
- Keep `harness/spec.md` lean
- After sprint 3, summarize all QA findings into `harness/build-status.md`
- Reference `build-status.md` instead of individual old QA reports
- Delete resolved qa-report files to reduce noise

**Simplification option (v2)**: For capable models, consider dropping sprint contracts
entirely — give the full spec and let the generator build continuously, with evaluator
checkpoints every N features instead of formal sprints. Test whether the overhead of
sprint negotiation is still load-bearing for your use case.

---

## Final Delivery

After all sprints pass:

1. Run `npm run build` one final time
2. Write `harness/final-summary.md`:
   - What was built
   - Known limitations / stubs
   - How to run
3. Present to user: "Voici ce qui a été construit. Veux-tu itérer sur une feature spécifique ?"
4. Offer to clean up `harness/` directory

---

## Common Failures & Fixes

| Failure | Fix |
|---|---|
| Generator under-scopes | Always run Planner first |
| Generator goes off-rails | Smaller sprints (2–3 features) |
| Evaluator too lenient | Reinforce "skeptical, not generous" + concrete FAIL examples |
| Evaluator finds bugs, Generator ignores | Reference qa-report by line in next prompt |
| App feels like a demo | Increase Product Depth weight + add AI feature sprint |
| Context confusion after many sprints | Write build-status.md summary |
| Build fails silently | Always run `npm run build` as first QA step |
