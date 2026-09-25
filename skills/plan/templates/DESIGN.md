# <NAME> — Design Doc

Every section is mandatory to consider, optional to fill: `N/A: <reason>` is a valid
answer, a blank section is not. Design effort scales with risk, not template completeness.

## Context
Why this work exists.

## Functional Requirements

### <a id="fr1"></a>FR1 — Short title
Description of what the system must do.

### <a id="fr2"></a>FR2 — Short title
Description.

## Non-Functional Requirements

Each NFR is a measurable scenario, not free text. A constraint without a measure
cannot enter the constitution because nothing can verify it.

### <a id="nfr1"></a>NFR1 — Short title
- **Scenario**: <stimulus> under <environment> → <expected response>
- **Measure**: <threshold, pass/fail — e.g. p99 < 200ms at 100 req/s>
- **Verify**: `<command that exits 0 when the measure holds>` — or `manual: <reason>`

## Non-goals
What this design explicitly does not address.

Before listing a non-goal, verify it is truly out of scope:
- **Already covered?** Check whether existing behavior, architecture, or a planned feature already provides the capability. If it does, it is not a non-goal — it is a fact to document (e.g., "OR composition is already provided by the first-match-wins evaluation order").
- **Deferred or excluded?** If it is genuinely not covered and not planned, state *why* it is excluded (cost, complexity, low priority) so the decision can be revisited later.
- **Misclassified FR?** If exploration reveals the capability is actually needed for the stated goals, promote it to an FR instead.

A non-goal that turns out to be already solved is a planning error — it signals incomplete analysis of the existing system.

## Rabbit holes
Areas of known uncertainty where unbounded time could be lost.
For each: state what to avoid and the constraint that caps exploration.

## Failure modes

What breaks and what happens then. Happy-path invariants live in the TASKS.md
transformations table; this table designs the error paths.

| Failure | Detection | Response | Blast radius |
|---|---|---|---|
| e.g. downstream API timeout | request > 2s | retry ×3, then degrade to cached value | single request |

Each row that yields a rule becomes an error-path invariant in the TASKS.md
transformations table, so error handling gets tests too.

## Design
Architecture, C4 diagrams (levels 1-2 via Mermaid), data model, interfaces.

- Mark **trust boundaries** on the C4 diagram. For each boundary a flow crosses,
  run a STRIDE pass and record the resulting constraints here or in an ADR.
- Add one Mermaid `sequenceDiagram` per critical flow — the class diagram shows
  structure, not ordering.

Decisions:
- [Decision title](./adrs/decision-name.md)

## Data & migration
`N/A: <reason>` when no persistence changes. Otherwise: schema delta, consistency
model, migration order (expand/contract), rollback path.

## Cross-cutting Concerns
Observability (SLIs for new paths), rollout (flags, canary), rollback.
