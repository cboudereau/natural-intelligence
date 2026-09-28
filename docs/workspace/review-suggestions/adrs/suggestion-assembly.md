---
status: proposed
---
# Suggestion assembly

Addresses: [FR2](../DESIGN.md#fr2)

## Problem

Who turns a finding into a suggestion block, and from what? The block must carry the replacement lines verbatim with exact indentation — neither platform normalises whitespace, and a mismatch renders the suggestion noisy or dead. Findings come from checklist subagents and [`ni:reviewer`](../../../../agents/reviewer.md) (haiku) as one-line prose.

## Options

| Option | Pros | Cons |
|---|---|---|
| Poster assembles: the agent that posts re-reads the target file and builds the block | Whitespace copied from the real file; extends the existing answering-flow rule ("read the actual file, never from comment text alone"); subagent contracts unchanged | One extra file read per suggestion |
| Reviewer subagents emit full payloads (code, span, indentation) | No second read | Haiku one-liner contract breaks; payload trusted without the file open — exactly what the existing rule forbids; every subagent needs the rule |
| Hybrid: subagent marks a span, poster assembles | Span hint saves a search | Two owners for one artifact; hint often stale after re-read anyway |

## Decision

Poster assembles. The posting step (main thread in the giving-a-review flow; the loop's posting step in [`review-loop`](../../../../commands/review-loop.md)) re-reads the target span from the working tree at the reviewed ref, copies the lines, applies the fix, and emits the block. The answering-flow rule generalises to one rule for both directions: no suggestion without the real file open. Subagent output contracts stay one-line prose.

## Consequences

- [`agents/reviewer.md`](../../../../agents/reviewer.md) needs no contract change; haiku stays viable.
- Each suggestion costs one targeted Read of the anchored span — bounded by findings count, no full-file dumps.
- The whitespace rabbit hole (DESIGN) is capped by construction: replacement lines are edited copies, never regenerated.
- One rule owner: the generalised read-first rule sits in [`code-review/SKILL.md`](../../../../skills/code-review/SKILL.md), referenced by both flows.
