# plan-skill-split — Design Doc

Amends: [20260928_gh-cli-integration](../../20260928_gh-cli-integration/README.md) (builds on the link-lint bullet that change put in Phase 4c).

## Context

[`skills/plan/SKILL.md`](../../../skills/plan/SKILL.md) is 352 lines against the 300-line limit set by [`skills/skill/SKILL.md:78`](../../../skills/skill/SKILL.md). The whole file loads on every trigger, but three blocks are read at one moment only: the Phase 4c pre-flight gate (lines 204–249), the Phase 5 autopilot detail (250–321), and the Phase 6 integration steps (322–341). The meta-skill's own rule: move heavy reference material to a sibling file and link it.

## Functional Requirements

### <a id="fr1"></a>FR1 — Pre-flight template
The Phase 4c gate (the three checklists: per task, constitution completeness, autopilot readiness — plus the plan-commit instruction) moves to a new template (templates/preflight.md). Phase 4c in SKILL.md shrinks to: copy the template into the workspace as PREFLIGHT.md, tick every box on disk, commit the plan. Disk-is-truth: a resumed session sees whether the gate passed instead of trusting memory. The link-lint bullet added by [20260928_gh-cli-integration](../../20260928_gh-cli-integration/README.md) moves with the checklist.

### <a id="fr2"></a>FR2 — Autopilot reference file
The Phase 5 detail (per-task contract, durability invariants, orchestrator loop, severity levels 1–3, discovery routing) and the Phase 6 integration steps with their commit template move to a new reference file (autopilot.md), following the reference-file conventions: title, first line names the owning skill, `REQUIRED BACKGROUND` back-link, no link to another reference file. Phase 5 and Phase 6 in SKILL.md shrink to a summary paragraph each plus the link.

### <a id="fr3"></a>FR3 — No content loss, no duplication
Every extracted rule appears in exactly one place. SKILL.md keeps: triggers, Goal (including the human-time-budget rationale — user-excluded from extraction), Delegation model, Structure, Document order, Constitution (including the within/outside lists — user-excluded), Phases 0–4b, phase summaries for 4c/5/6, Cross-referencing, Amending.

## Non-Functional Requirements

### <a id="nfr1"></a>NFR1 — Size limit met
- **Scenario**: after the split → SKILL.md within the meta-skill's limit
- **Measure**: [`skills/plan/SKILL.md`](../../../skills/plan/SKILL.md) under 300 lines; both new files under 300 lines
- **Verify**: `awk 'END{exit NR>=300}' skills/plan/SKILL.md && awk 'END{exit NR>=300}' skills/plan/autopilot.md && awk 'END{exit NR>=300}' skills/plan/templates/preflight.md`

### <a id="nfr2"></a>NFR2 — Plugin validates
- **Scenario**: after every commit → plugin valid
- **Measure**: `claude plugin validate .` exits 0
- **Verify**: `claude plugin validate .`

## Non-goals

- Changing any planning rule or phase semantics — this is a move, not a rewrite. Wording may only tighten to lite prose where a sentence is split by the move.
- Renumbering phases or renaming anchors other skills might use.
- Bumping the plugin version: 1.4.0 is still unreleased on this branch.

## Rabbit holes

- Rewriting the autopilot content while moving it. Cap: cut and paste with only section headers and back-links added; any rule change is out of scope.

## Failure modes

| Failure | Detection | Response | Blast radius |
|---|---|---|---|
| A rule lost in the move | task verify greps key phrases in exactly one file | restore from git history | one skill |
| A stale in-file anchor link to a moved section | grep internal `#phase-` anchors after the move | repoint to the new file | links only |

## Design

One decision, ratified by the user: [Pre-flight as a workspace-copied template](./adrs/preflight-as-template.md).

Resulting layout:
- [`skills/plan/SKILL.md`](../../../skills/plan/SKILL.md) (~230 lines): phases 0–4b full, 4c/5/6 as summary plus link.
- skills/plan/templates/preflight.md (new): tickable gate checklist, copied to the workspace as PREFLIGHT.md.
- skills/plan/autopilot.md (new): Phase 5 contract, invariants, loop, severities, discovery; Phase 6 steps and commit template.

## Data & migration

N/A: markdown-only.

## Cross-cutting Concerns

Rollout: none beyond the commit (version already bumped on this branch). Rollback: git revert.
