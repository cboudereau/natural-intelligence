---
status: accepted
---
# Pre-flight as a workspace-copied template

Addresses: [FR1](../DESIGN.md#fr1)

## Problem

Where does the extracted Phase 4c gate live: a read-only reference file, or a template copied into each workspace? Today the gate's ticks live only in conversation, so a resumed session cannot see whether pre-flight actually passed.

## Options

| Option | Pros | Cons |
|---|---|---|
| Reference file (read-only checklist) | Smallest change | Ticks still live in conversation; resume cannot audit the gate |
| Template copied to the workspace as PREFLIGHT.md, ticked on disk | Disk-is-truth like TASKS.md; resume sees gate state; dies with the workspace at Phase 6 | One more file per workspace |

## Decision

Template, copied to the workspace and ticked on disk (user decision, 2026-09-28). It joins the existing templates (DESIGN, TASKS, adr) and follows their linking conventions.

## Consequences

- Phase 0 resume can check PREFLIGHT.md alongside TASKS.md.
- Phase 6 teardown deletes it with the workspace; git history preserves it.
- The link-lint bullet ships inside the template, so every future workspace runs it mechanically.
