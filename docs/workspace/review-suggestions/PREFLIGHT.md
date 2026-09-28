# review-suggestions — Pre-flight gate

Copied from [templates/preflight.md](../../../skills/plan/templates/preflight.md); ticked on disk 2026-09-28.

**Per task**:
- [x] Types it creates or modifies reference the domain model (artifact traceability table — markdown-only change, no code types)
- [x] Constraints are extracted from ADRs, invariants, and transformation rules
- [x] Tests are defined — named, with expected behavior described (red greps confirmed: `suggestion:-0+0` present, `batch_apply` absent, `startLine` absent, loop has no "suggestion" and no auto-resolution wording, "own thread" absent from the skill, version 1.4.0)
- [x] Verify command is copy-pasteable and exits 0 on success
- [x] Acceptance criteria are pass/fail with no subjective language
- [x] Time-box is set (max 50 min, none over 90)
- [x] Dependencies on other tasks are declared
- [x] Every planned domain type appears in the requirement traceability table (all four artifacts traced)
- [x] Task is `downhill` — no uncertainty remains (payloads researched with sources; unverifiable parts flagged `manual`)

**Constitution completeness**:
- [x] Domain model covers every planned domain type in diagram and traceability table (N/A diagram: instruction files, not code; the traceability table is the model)
- [x] Every type in the traceability table maps to at least one FR or NFR
- [x] Every traceability row has a stability value (`internal` or `published`); each `published` row has a compatibility rule (all published; loop stays agnostic, syntax containment, rule ownership)
- [x] Every NFR has a measure and a verify command (or an explicit `manual: <reason>`)
- [x] Transformations table covers every function that enforces a domain rule (instruction invariants table)
- [x] Every failure-modes row that yields a rule appears as an error-path invariant in the transformations table (stale anchor, deleted lines, rate limits)
- [x] Every DESIGN.md section is filled or marked `N/A: <reason>` — no blank sections
- [x] No constraint is ambiguous enough that two reasonable agents would interpret it differently
- [x] Link lint green — every file reference in workspace docs is a clickable link:
  `! grep -rPn '(?<!\[)\x60(?:[\w.-]+/)*[\w.-]+\.md(?::\d+(?:[-,:]\d+)?)?\x60' docs/workspace/review-suggestions --include='*.md'` — verified, exit 0

**Autopilot readiness**:
- [x] Build, test, and lint commands pass (green baseline) — `claude plugin validate .` exit 0 confirmed
- [x] Known-failing tests are explicitly listed with their reason (`--strict` on the transient root CLAUDE.md anchor)
- [x] Every session has a `Skills` field — `skill`, `git-conventions`, `evidence-based-analysis` all exist
- [x] Session checkpoints are defined and ordered
- [x] Total estimated time fits within the target session window (~3.5H)

If any item fails, fix it before proceeding. — none failed.

**Plan commit**: gate passed; committing all workspace artifacts.
