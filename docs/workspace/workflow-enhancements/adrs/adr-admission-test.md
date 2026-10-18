---
status: accepted
---
# ADR admission test

Addresses: [FR1](../DESIGN.md#fr1), [FR2](../DESIGN.md#fr2)

## Problem

Phase 3 of the complex route lists eight kinds of decision that "warrant an ADR" (hard-to-reverse choices, trade-offs, inherited constraints, rejected alternatives, conventions, versioning, security constraints from STRIDE). The list reads as a quota: on the benchmark's complex planning scenario the route produced four ADRs for a task with two hard-to-reverse decisions, and every ADR costs a human ratification (Phase 3 budget, ~10 min) plus tokens in every later session that reads the workspace. Conventions and inherited constraints are facts to record in DESIGN.md, not decisions to ratify.

## Options

| Option | Pros | Cons |
|---|---|---|
| A. Keep the list, add "at most N ADRs" | Simple cap | Arbitrary; a large system can need five, a flag needs none |
| B. One admission test: hard to reverse, surprising later, real alternatives weighed; all three required | Each ADR earns its ratification; the test is checkable by a reviewer; the eight kinds survive as examples | Some conventions lose their ADR and move to DESIGN.md Design or the glossary |
| C. ADRs only for persistence and public contracts (the small route's rule) | Fewest ADRs | Misses protocol, security, and inherited-constraint decisions on large systems |

## Decision

**B.** Phase 3 opens with the test. A decision that fails any of the three goes to DESIGN.md (Design section for conventions, Context for inherited constraints) or to the glossary when it is a naming choice. The STRIDE constraint item stays an example: it passes the test only when a boundary is crossed, which FR2 ties to the diagram requirement. The small route's rule (persistence or wire protocol) is a special case of the test and stays as written.

## Consequences

Easier: fewer ADRs per workspace, each one a decision a reader would otherwise question; the Phase 3 human budget shrinks on small systems. Harder: the agent must argue the three criteria in the ADR's Problem section; a reviewer can reject an ADR that lists no alternative.
