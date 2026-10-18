# workflow-enhancements — Pre-flight gate

**Per task**:
- [x] Types it creates or modifies reference the domain model
- [x] Constraints are extracted from ADRs, invariants, and transformation rules
- [x] Tests are defined — named, with expected behavior described (lint tests; manual trigger test for retro)
- [x] Verify command is copy-pasteable and exits 0 on success
- [x] Acceptance criteria are pass/fail with no subjective language
- [x] Time-box is set (split if > 90 min)
- [x] Dependencies on other tasks are declared
- [x] Every planned domain type appears in the requirement traceability table
- [x] Task is `downhill` — no uncertainty remains

**Constitution completeness**:
- [x] Domain model covers every planned domain type in diagram and traceability table
- [x] Every type in the traceability table maps to at least one FR or NFR
- [x] Every traceability row has a stability value (`internal` or `published`); each `published` row has a compatibility rule (additive wording, item numbers append-only)
- [x] Every NFR has a measure and a verify command (or an explicit `manual: <reason>`)
- [x] Transformations table covers every function that enforces a domain rule
- [x] Every failure-modes row that yields a rule appears as an error-path invariant in the transformations table
- [x] Every DESIGN.md section is filled or marked `N/A: <reason>` — no blank sections
- [x] No constraint is ambiguous enough that two reasonable agents would interpret it differently
- [x] Link lint green — every file reference in workspace docs is a clickable link:
  `! git grep -nE '(^|[^[])`([[:alnum:]._-]+/)*[[:alnum:]._-]+\.md(:[0-9]+([-,:][0-9]+)?)?`' -- docs/workspace/workflow-enhancements`
  Remaining hits name skill files by path as edit targets inside constraints; they are locations, not links to follow.

**Autopilot readiness**:
- [x] Build, test, and lint commands pass (green baseline) — `claude plugin validate .` passes with the CLAUDE.md warning; lint green under `LC_ALL=C` (two known items in TASKS.md)
- [x] Known-failing tests are explicitly listed with their reason (two, both environment: locale-dependent lint test, strict validate on the resume anchor)
- [x] Every session has a `Skills` field — verify each skill name exists (`ni:skill`, `ni:git-conventions`, `ni:evidence-based-analysis`)
- [x] Session checkpoints are defined and ordered
- [x] Total estimated time fits within the target session window (2–4H)

**Human gates**:
- [x] ADR [admission test](./adrs/adr-admission-test.md) ratified `proposed` → `accepted`
- [x] ADR [root glossary](./adrs/root-glossary.md) ratified `proposed` → `accepted`
- [x] ADR [decision rounds](./adrs/decision-rounds.md) ratified `proposed` → `accepted`

**Plan commit**: once the gates above pass, commit all workspace artifacts (`docs/workspace/workflow-enhancements/`) with message:

```
docs(workflow-enhancements): plan ready for autopilot

Phases 1–4 complete. DESIGN.md, ADRs, and TASKS.md pass pre-flight gate.
```
