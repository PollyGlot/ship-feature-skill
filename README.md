# ship-feature-skill

A Claude Code skill that enforces a disciplined workflow for shipping non-trivial features.

## What it does

`/ship-feature` chains seven phases with explicit STOP gates between each, preventing premature implementation and ensuring the user validates every important step:

1. **Grill** — sharpen scope, constraints, alternatives, success criteria
2. **Prototype** (if UI/UX) — produce 2-3 radically different variants before committing
3. **ADR** (if architectural) — document the structural decision
4. **PRD + tasks** — synthesize plan, split into individually-grabbable tasks
5. **TDD** — red → green → refactor, formatter + tests green before every commit
6. **PR** — feature branch, **never** auto-merge, manual merge after CI green
7. **Close subject** — archive with short recap

## Install

Install via the [`skills`](https://skills.sh) CLI:

```bash
npx skills@latest add PollyGlot/ship-feature-skill -g --all -y
```

## When to use

- Non-trivial feature (>1 file, UX impact, DB/API migration)
- You want a durable PRD/ADR, not just a patch
- **Not for**: trivial bugfix, mechanical refactor, typo, doc-only changes

## Why it exists

This skill encodes recurring frictions observed in real Claude Code sessions:

- Claude jumping to implementation before plan approval
- Auto-merging PRs against project rules
- Presenting audit findings without verifying them against the actual codebase

Each non-negotiable rule and anti-pattern in `skills/ship-feature/SKILL.md` traces back to one of these.

## License

MIT
