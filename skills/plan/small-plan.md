# Small plan — one document

Reference file of the [`plan`](SKILL.md) skill. Route: single-session task, no
multi-session trigger (see the triage in SKILL.md).

## Format

One markdown file, four sections, nothing else. No PREFLIGHT.md, no adrs/ folder, no
Mermaid diagram. A hard-to-reverse decision, if one appears, gets an ADR file; every
other choice is an inline bullet. This applies to build tasks too: a small
implementation task records its choices as bullets, not ADR files.

```markdown
# <task> — plan

## Goal
One or two sentences: what exists when this is done, and how it is verified.

## Decisions
- <choice>: <option taken> — <reason in one clause>

## Tasks
- [ ] <task> — test: `<named test>` — verify: `<command>`
- [ ] <task> — test: `<named test>` — verify: `<command>`
Resume: continue at the first unchecked task; re-run the last verify before trusting state.

## Out of scope
- <explicitly excluded item> (one line each, only when the brief invites scope creep)
```

## Rules

- The `## Tasks` section is the machine layer: checkboxes, named tests, verify
  commands, the resume line. A few dozen words protect the crash, rate-limit, and
  compaction cases that the full TASKS.md protects on multi-session work.
- Plan exactly what the brief asks. Extras become one out-of-scope line, never
  sections or tasks.
- Code snippets stay illustrative — shapes and signatures, not implementations.
- Tick boxes as tasks complete; the document is the progress tracker.

## Escalation

A small plan that grows a second hard-to-reverse decision, a domain model, or a
second session of work has outgrown this route: re-run the triage in
[`plan`](SKILL.md), switch to the complex route, and carry the content over. Say so
in one line; never maintain both formats in parallel.
