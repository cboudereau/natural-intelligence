---
status: accepted
---
# Root glossary

Addresses: [FR6](../DESIGN.md#fr6)

## Problem

The complex route builds a domain model in `TASKS.md` (class diagram, traceability table) and deletes it with the workspace at Phase 6. The dated design doc keeps the diagram, but no file holds the project's vocabulary in a form every later session reads before exploring. Each session re-derives the terms from code, spends tokens on synonyms, and names new things inconsistently. A shared vocabulary is also the cheapest terseness lever: a term that means one thing costs one token.

## Options

| Option | Pros | Cons |
|---|---|---|
| A. Keep the vocabulary in the dated design doc | No new file | Buried in one feature's doc; other features do not read it |
| B. Root `GLOSSARY.md`, owned by `plan`, written at Phase 6, read by onboarding and Phase 4a | One durable file, one owner, read on every task through onboarding | One more root file; must stay a glossary, not a spec |
| C. Per-skill reading of all dated design docs | No new file | Unbounded reading cost; grows with every feature |
| D. Glossary inside root `CLAUDE.md` | Always loaded | Loaded into every prompt, including sessions that never touch the domain; `CLAUDE.md` is the resume anchor, not a reference |

## Decision

**B.** Format: a markdown list, one entry per term, `**Term**: definition in one or two sentences. Replaces: <words not to use>.` Terms sorted alphabetically. No types, functions, or file paths: implementation detail lives in the design docs. Writers: Phase 6 integrate adds or updates the terms the workspace resolved, and the `retro` skill may propose an entry when a session misnamed something. Readers: `software-engineer/onboarding.md` Repository conventions ("read `GLOSSARY.md` when present"), Phase 4a domain-model axis. Absent file: every reader proceeds silently; nothing outside a plan workspace creates it.

## Consequences

Easier: consistent names across sessions and features; the Phase 4a domain-model axis starts from an agreed vocabulary; terse replies without re-explaining terms. Harder: Phase 6 gains one step; a reviewer must keep the file a glossary (the format line is the test).
