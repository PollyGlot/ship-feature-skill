---
name: ship-feature
description: Strict workflow for shipping a non-trivial feature end-to-end. Chains grill → prototype (if UI/UX) → ADR → PRD → TDD → PR → close with explicit stop-gates between phases. Use for changes that deserve a formal plan (multi-file features, UX impact, migration). DO NOT use for trivial bugfix, mechanical refactor, or typo.
---

# Ship Feature

Opinionated workflow to ship a complete feature with discipline. Forces stops between phases for user validation. Addresses recurring frictions: premature implementation, forbidden auto-merge, unverified findings.

## When to use

- Non-trivial feature (>1 file, UX impact, DB/API migration)
- You want a durable PRD/ADR, not just a patch
- **Not for**: trivial bugfix, mechanical refactor, typo, doc-only changes

## Process

### 1. Grill (sharpening)

Run `/grill-me` (or an equivalent grilling skill), or ask 5-7 targeted questions on:
- **Scope**: in/out, what we will NOT do
- **Constraints**: perf, accessibility, migration, platforms
- **Discarded alternatives**: why
- **Success criteria**: measurable, verifiable

> **STOP**. Wait for user validation on the answers before continuing.

### 2. Prototype (if UI/UX)

**Trigger**: the feature touches UI/UX (new screen, redesign, micro-interaction, navigation, visible widget).

Run `/prototype` (or equivalent) to produce **2-3 radically different variants** (layout, hierarchy, motion, density). Toggleable from a single route for fast comparison.

Divergence criteria:
- **Not two variants of the same theme**: if V1 and V2 only differ by padding, you missed the point
- Cover at least 2 directions (e.g. minimalist vs dense, static vs animated)
- Respect the project's design doc and anti-references

Deliverable: screenshot/route + one paragraph per variant (strengths, weaknesses, anti-pattern avoided).

> **STOP**. Wait for the user to **choose the variant** before continuing. The chosen variant frames phases 3-5.

### 3. ADR (if architectural decision)

If the feature involves a structural choice (new dependency, pattern, DB schema, platform), write an ADR in `docs/adr/NNNN-{slug}.md`.

Format: `Context` · `Decision` · `Consequences` · `Alternatives`.

> **STOP**. Wait for `go ADR` before continuing.

### 4. PRD + tasks

- Use `/to-prd` (or equivalent) to synthesize a PRD in the project's tasks/plans directory
- Use `/to-issues` (or equivalent) to split into individually-grabbable tasks

> **STOP**. Wait for explicit `go implementation` before **any Edit/Write on production code**.

### 5. TDD (red → green → refactor)

Per task:
1. Write the failing test — the project's test runner confirms the red
2. Implement the minimum to pass (green)
3. Refactor if needed
4. Project formatter + test suite must pass before commit
5. Commit with Conventional Commits in English (`type(scope): subject`)

### 6. PR

- **Before creating the PR, run `/simplify`** to review changed code for reuse, quality, and efficiency, then fix any issues found. Only proceed to PR creation once `/simplify` has been run and the resulting cleanups (if any) are committed.
- Create a feature branch and open a PR towards the project's main dev branch
- **NEVER auto-merge** (`gh pr merge --auto` forbidden unless the project explicitly allows it)
- Wait for CI green
- Wait for user approval
- Merge manually

### 7. Close subject

- Archive the closed subject (use `/close-subject` or equivalent)
- Short recap in `_recap.md`

## Non-negotiable rules

- **No production code without explicit PRD approval** (gate phase 4 → 5)
- **For UI/UX features, prototype 2-3 variants before the PRD** (gate phase 2 → 3) — no PRD on an unchosen direction
- **Formatter + tests green** before every commit
- **Always run `/simplify` before opening a PR** (gate phase 5 → 6) — no PR without a simplification pass
- **Never auto-merge**
- **Verify every finding before asserting it** (lockfile, code, spec) — mark "to investigate" rather than asserting blindly

## Anti-patterns

- Starting to code during the grill or PRD phase "to save time"
- Skipping the prototype phase on a UI/UX feature with "I already see the solution"
- Presenting 2-3 variants that only differ by details (real divergence required)
- Running `gh pr merge --auto` because CI is "probably OK"
- Pushing without running the formatter because "the hook will do it"
- Presenting an audit without grepping the code first
- Opening the PR without running `/simplify` first because "the diff looks clean"
