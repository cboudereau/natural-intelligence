---
status: accepted
---
# Decision rounds

Addresses: [FR7](../DESIGN.md#fr7)

## Problem

Phase 3 asks the human to make the hard decisions the agent cannot, inside a ~10 minute budget, by reading draft ADRs one at a time. When several decisions are open the human answers them across several messages, each costing a full round trip, and decisions that depend on earlier answers are drafted before those answers exist. In delegated or headless contexts the route already decides alone (SKILL.md "decide, do not ask"); the interactive case has no equivalent discipline.

## Options

| Option | Pros | Cons |
|---|---|---|
| A. Status quo: one ADR at a time, human reads and ratifies | Simple | Serial round trips; dependent decisions drafted on guesses |
| B. Decision round: list every decision whose prerequisites are settled, numbered, each with a recommended answer phrased so "yes" accepts it; next round after the answers; stop when nothing is open | Batches the human's attention; dependencies respected by construction; "yes" is the fast path | A new sub-step in Phase 3; needs a clear exit to the delegated rule |
| C. Interview on every task, inside the small route too | Maximum alignment | Noise on small tasks; the small route is one document by design |

## Decision

**B**, inside Phase 3 of the complex route only. Shape: a numbered list; each item names the decision, the options in one line, and the agent's recommended answer with its reason; the human replies with numbers and "yes", or an alternative. A decision whose answer depends on an open item waits for the next round. Accepted answers go to DESIGN.md, or to an ADR at `proposed` when they pass the admission test. The round is skipped under the existing delegated-context rule, which the Phase 3 text names first.

## Consequences

Easier: fewer messages per plan; the human's Phase 3 budget covers several decisions at once; dependent decisions are drafted on real answers. Harder: the agent must know which decisions are open before drafting any ADR, so Phase 2 must surface them; the round text must stay short or it costs more than it saves.
