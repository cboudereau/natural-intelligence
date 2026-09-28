# plan-skill-split — Design Doc (seed)

Status: seed only — Phase 2 starts after [gh-cli-integration](../../20260928_gh-cli-integration/README.md) integrated. Scope was fixed by the user on 2026-09-28.

## Context

[`skills/plan/SKILL.md`](../../../skills/plan/SKILL.md) is 350 lines against the 300-line limit set by [`skills/skill/SKILL.md:78`](../../../skills/skill/SKILL.md). The whole file loads on every trigger; the blocks below are read at one moment only.

## Scope (user decision, 2026-09-28)

Externalise:
- Phase 4c pre-flight gate → a new template (templates/preflight.md), copied into the workspace as PREFLIGHT.md at Phase 4c and ticked on disk (disk-is-truth: a resumed session sees whether the gate passed). Includes the link-lint bullet added by gh-cli-integration.
- Phase 5 autopilot detail (per-task contract, durability invariants, orchestrator loop, severity levels, discovery routing) → a new reference file (autopilot.md).
- Phase 6 integration steps and commit template → the same autopilot.md reference file.

Explicitly excluded (stay in SKILL.md, user decision):
- Goal section's human-time-budget rationale.
- Constitution detail (the within/outside bullet lists).

Constraints: reference files one level deep, back-link to SKILL.md, `REQUIRED BACKGROUND` line, no reference file links another reference file. Target: SKILL.md under 300 lines. Amends the plan skill only; behaviour unchanged.
