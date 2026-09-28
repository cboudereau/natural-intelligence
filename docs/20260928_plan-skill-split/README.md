# plan-skill-split

Split of [`skills/plan/SKILL.md`](../../skills/plan/SKILL.md) from 352 to 228 lines, under the 300-line limit set by [`skills/skill/SKILL.md`](../../skills/skill/SKILL.md). Pure move, no rule changed. Integrated 2026-09-28. Amends [20260928_gh-cli-integration](../20260928_gh-cli-integration/README.md).

## Design

- [plan-skill-split](designs/plan-skill-split.md) — what moved, what stayed, no-loss verification

## Decisions (ADRs, accepted)

- [Pre-flight as a workspace-copied template](adrs/preflight-as-template.md) — the gate is copied to the workspace as PREFLIGHT.md and ticked on disk

## What changed in the plugin

- [`skills/plan/templates/preflight.md`](../../skills/plan/templates/preflight.md) (new, 45 lines): the Phase 4c gate — three checklists (link lint included) and the plan-commit instruction
- [`skills/plan/autopilot.md`](../../skills/plan/autopilot.md) (new, 97 lines): Phase 5 detail (per-task contract, durability invariants, orchestrator loop, severities, discovery) and Phase 6 steps with the integration commit template
- [`skills/plan/SKILL.md`](../../skills/plan/SKILL.md): 228 lines; Phases 4c, 5 and 6 are summaries plus links; Phase 0 resume also checks PREFLIGHT.md
