# plan-skill-split — Tasks

Design: [DESIGN.md](./DESIGN.md)

## Analysis

Build: N/A — markdown-only plugin
Test: `claude plugin validate .` — verified green (known transient root [`CLAUDE.md`](../../../CLAUDE.md) warning, this workspace's own anchor)
Lint: link lint (see checkpoint)

Section map of [`skills/plan/SKILL.md`](../../../skills/plan/SKILL.md) (352 lines): Phase 4c = lines 204–249, Phase 5 = 250–321, Phase 6 = 322–341. Extraction ≈ 138 lines out, ≈ 15 summary lines back in → ≈ 229 lines.

### Known-failing tests
| Test | Reason | Action |
|---|---|---|
| `claude plugin validate . --strict` | Transient workspace [`CLAUDE.md`](../../../CLAUDE.md) at plugin root | strict gate at this workspace's Phase 6 teardown |

### Requirement traceability

| Artifact | Stability | Addresses | Notes |
|---|---|---|---|
| `skills/plan/templates/preflight.md` | published | [FR1](./DESIGN.md#fr1) | Copied per workspace as PREFLIGHT.md |
| `skills/plan/autopilot.md` | published | [FR2](./DESIGN.md#fr2) | Reference file, linked from Phases 5–6 |
| [`skills/plan/SKILL.md`](../../../skills/plan/SKILL.md) | published | [FR1](./DESIGN.md#fr1), [FR2](./DESIGN.md#fr2), [FR3](./DESIGN.md#fr3) | Anchors and phase names stay |

### Instruction invariants

| Rule | Where | Invariant |
|---|---|---|
| One owner | all three files | Every extracted rule appears in exactly one file; SKILL.md carries a summary line, never the rule text |
| Gate on disk | Phase 4c summary | Copy template → workspace PREFLIGHT.md → every box ticked before autopilot → plan commit |
| Reference conventions | new files | Back-link to SKILL.md, `REQUIRED BACKGROUND` line, no reference-to-reference link |
| Scope freeze | SKILL.md | Goal time-budget rationale and constitution within/outside lists stay (user exclusion) |

## Tasks

### 1. Extract pre-flight template and autopilot reference ([FR1](./DESIGN.md#fr1), [FR2](./DESIGN.md#fr2), [FR3](./DESIGN.md#fr3), [NFR1](./DESIGN.md#nfr1))
**Goal**: [`skills/plan/SKILL.md`](../../../skills/plan/SKILL.md) under 300 lines with no rule lost.
**Artifacts**: `skills/plan/SKILL.md`, `skills/plan/templates/preflight.md` (new), `skills/plan/autopilot.md` (new)
**Constraints**:
- [ADR: preflight-as-template](./adrs/preflight-as-template.md) — template copied to the workspace, ticked on disk, deleted at Phase 6
- Cut-and-paste move; only section headers, back-links, and the copy instruction are new text (DESIGN rabbit hole cap)
- Phase 0 resume gains one clause: check PREFLIGHT.md state alongside TASKS.md
- Phase 6 teardown list gains PREFLIGHT.md deletion (dies with the workspace)
- User exclusions respected: Goal rationale and constitution lists stay in SKILL.md
**Tests** (red before the edit):
- `test -f skills/plan/templates/preflight.md` fails — must pass after
- `test -f skills/plan/autopilot.md` fails — must pass after
- `awk 'END{exit NR>=300}' skills/plan/SKILL.md` fails (352 lines) — must pass after
**Verify**: `awk 'END{exit NR>=300}' skills/plan/SKILL.md && awk 'END{exit NR>=300}' skills/plan/autopilot.md && awk 'END{exit NR>=300}' skills/plan/templates/preflight.md && grep -c "Per-task contract" skills/plan/autopilot.md | grep -qx 1 && ! grep -q "Per-task contract" skills/plan/SKILL.md && grep -q "severity" skills/plan/autopilot.md && grep -q "Integration commit" skills/plan/autopilot.md && ! grep -qi "integration commit" skills/plan/SKILL.md && grep -q "link lint" skills/plan/templates/preflight.md && grep -q "PREFLIGHT.md" skills/plan/SKILL.md && claude plugin validate .`
**Acceptance criteria**:
- [ ] SKILL.md under 300 lines; both new files exist and are under 300
- [ ] Extracted rules present exactly once (verify greps green)
- [ ] Phase 0 and Phase 6 wired to PREFLIGHT.md
- [ ] Verify command exits 0
**Depends on**: (none)
**Time-box**: ~60 min

## Sessions

### Session 1 — Plan skill split (~1H)
Tasks: 1
**Skills**: `skill` (ni conventions), `git-conventions` (commits)
**Checkpoint**: task 1 verify command, plus the link lint: `! grep -rPn '(?<!\[)\x60(?:[\w.-]+/)*[\w.-]+\.md(?::\d+(?:[-,:]\d+)?)?\x60' docs/workspace/plan-skill-split --include='*.md'`
**Commit point**: yes

## Quality gates (post-session review)
- [ ] Acceptance criteria green
- [ ] Diff review: pure move — no rule text changed, only headers/links/copy instruction added
- [ ] Organization: reference files one level deep; template joins templates/; back-links present
- [ ] Cross-referencing: link lint green
- [ ] Security: N/A — no auth content moved
