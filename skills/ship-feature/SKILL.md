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

Run `/grill-with-docs` — it grills against the project's existing domain language, CONTEXT.md, and ADRs, which is sharper than a context-free grill. Fall back to `/grill-me` only if no docs exist.

Target axes:
- **Scope**: in/out, what we will NOT do
- **Constraints**: perf, accessibility, migration, platforms
- **Discarded alternatives**: why
- **Success criteria**: measurable, verifiable

**This is a loop, not a single pass.** Keep grilling — new question → user answer → next sharper question — until **one** of these:
- The user says "stop", "on continue", "go", or equivalent
- You genuinely have no more sharpening questions (every axis is resolved). If you stop on this branch, say so explicitly ("no more questions on scope/constraints/alternatives/success — ready to move on") so the user can either confirm or surface a missed angle.

> **STOP**. Wait for user validation before continuing.

### 2. Prototype (if UI/UX)

**Trigger**: the feature touches UI/UX (new screen, redesign, micro-interaction, navigation, visible widget).

Run `/prototype` to produce **2-3 radically different variants** (layout, hierarchy, motion, density). Implementation rules for this project:

- **Single-screen throwaway**: all variants live on one `main.dart` entry point, toggleable from a single route — no integration into the real app yet.
- **Auto-launch**: after writing the prototype, launch it automatically (`flutter run` on the default device/simulator) so the user can interact immediately. Don't wait to be asked to run it.
- **Disposable**: this code is meant to be thrown away once a variant is chosen; don't over-engineer.

Divergence criteria:
- **Not two variants of the same theme**: if V1 and V2 only differ by padding, you missed the point
- Cover at least 2 directions (e.g. minimalist vs dense, static vs animated)
- Respect the project's design doc and anti-references

Deliverable: running `main.dart` with toggleable variants + one paragraph per variant (strengths, weaknesses, anti-pattern avoided).

> **STOP**. Wait for the user to **choose the variant** before continuing. The chosen variant frames phases 3-5.

### 3. ADR (if architectural decision)

If the feature involves a structural choice (new dependency, pattern, DB schema, platform), write an ADR in `docs/adr/NNNN-{slug}.md`.

Format: `Context` · `Decision` · `Consequences` · `Alternatives`.

**Autonomy rule**: do this phase on your own. Only ask the user when:
- A real ambiguity remains after re-reading CONTEXT.md / existing ADRs / the grill answers
- You're choosing between two alternatives both defensible and the call is the user's to make (cost, vendor lock-in, deadline trade-off)

Otherwise: write the ADR, mention it ("wrote ADR NNNN-{slug}"), and move on. No mandatory STOP — but flag the file so the user can review when they want.

### 4. PRD + tasks

- Use `/to-prd` (or equivalent) to synthesize a PRD in the project's tasks/plans directory
- Use `/to-issues` (or equivalent) to split into individually-grabbable tasks

> **STOP**. Wait for explicit `go implementation` before **any Edit/Write on production code**.

### 5. Implementation (solo or orchestrated)

#### 5a. Choose a mode

After `/to-issues`, look at the task list and decide:

- **Solo** — single task, tightly coupled tasks, or strict sequential dependencies. Implement directly in the current worktree.
- **Orchestrated** — 2+ tasks that are independent (no shared files, no ordering constraint). You become the **orchestrator**; spawn one sub-agent per task, each in its own worktree.

Default = solo when in doubt. Orchestrate only when parallelism actually buys time.

#### 5b. Per-task loop (both modes)

For each task:

1. Decide whether `/tdd` applies. Use it when the task has real logic, edge cases, or regression risk (red → green → refactor). Skip for pure scaffolding, copy-only changes, trivial migrations — and name the skip ("skipping `/tdd` because pure scaffolding").
2. Write the failing test first (if using TDD) — the project's test runner confirms the red.
3. Implement the minimum to pass (green).
4. Refactor if needed.
5. Project formatter + test suite must pass before commit.
6. Commit with Conventional Commits in English (`type(scope): subject`).

#### 5c. Orchestration mode (multi-worktree)

When you choose "orchestrated":

- **Spawn one sub-agent per independent task** with the `Agent` tool, `isolation: "worktree"`. Each sub-agent gets:
  - The task description and acceptance criteria from `/to-issues`
  - The branch naming convention (e.g. `feature/<slug>-<task-id>`)
  - Instruction: "When done, commit & push your worktree branch; do NOT open a PR; report a short summary"
- **Your role as orchestrator** (the parent agent):
  - **Don't write production code yourself.** You supervise. Exceptions: conflict resolution and integration glue.
  - **Track progress.** The SDK notifies you when a sub-agent finishes; poll only if you must.
  - **Review each sub-agent's output** before merging: read the diff, confirm tests pass on its branch, confirm it matches the task.
  - **Unblock & correct.** If a sub-agent is stuck or off-track, send a corrective message (`SendMessage`) or restart it with a sharper prompt. Don't silently fix its work.
  - **Merge into the integration worktree** (yours): once a sub-agent reports done and you've validated it, merge its branch into yours. Resolve conflicts here, not in the sub-agent's worktree.
- **Quality gates** (run on the integration worktree, not per sub-agent):
  - All sub-agent branches merged
  - Formatter / lint / typecheck green
  - Full test suite green
  - `/simplify` decision made once (per phase 6 rules), not once per sub-agent
- **End state**: integration branch is the single source of truth heading into phase 6. Sub-agent branches are disposable.

### 6. PR

- **Before creating the PR, decide whether to run `/simplify`** to review changed code for reuse, quality, and efficiency. Default = run it. Skip only when the diff is genuinely trivial (typo, copy tweak, one-line fix, mechanical rename) or when the workflow is a fast-path like `/push-pr-dev` that intentionally bypasses heavy review. If you skip, say so explicitly in the turn ("skipping `/simplify` because X"). If you run it, commit any resulting cleanups before opening the PR.
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
- **Use `/tdd` when there is real logic / edge cases / regression risk** — skip for pure scaffolding or copy-only changes, and announce the skip
- **In orchestrated mode, the orchestrator does not write production code** — supervise, review, merge, resolve conflicts; everything else goes through sub-agents
- **Sub-agents never open PRs** — only the orchestrator does, after integration
- **Default to running `/simplify` before opening a PR** (gate phase 5 → 6) — skip only for trivial diffs or fast-path workflows (`/push-pr-dev`, etc.), and announce the skip with a reason
- **Never auto-merge**
- **Verify every finding before asserting it** (lockfile, code, spec) — mark "to investigate" rather than asserting blindly

## Anti-patterns

- Starting to code during the grill or PRD phase "to save time"
- Skipping the prototype phase on a UI/UX feature with "I already see the solution"
- Presenting 2-3 variants that only differ by details (real divergence required)
- Running `gh pr merge --auto` because CI is "probably OK"
- Pushing without running the formatter because "the hook will do it"
- Presenting an audit without grepping the code first
- Skipping `/simplify` silently — if you skip, name the reason (trivial diff, fast-path workflow); silent skips erase the gate
- Orchestrator jumping in to write production code instead of delegating to a sub-agent
- Sub-agents opening their own PRs (only the orchestrator does, after integration)
- Orchestrating tasks that aren't actually independent — parallel work on shared files just produces merge conflicts
