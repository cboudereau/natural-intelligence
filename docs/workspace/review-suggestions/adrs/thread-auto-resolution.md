---
status: accepted
---
# Thread auto-resolution on re-review

Addresses: [FR6](../DESIGN.md#fr6)

## Problem

When the reviewee fixes a finding with a new commit instead of clicking the suggestion, the thread stays open. The reviewer returns, sees the fix, but nothing authorises resolving the thread — so consensus stalls on already-fixed findings. The user grants the autonomy: resolve own threads whose finding a new commit fixes (user decision, 2026-09-28).

## Options

| Option | Pros | Cons |
|---|---|---|
| Auto-resolve own threads after evidence check | Fixed findings close themselves; reviewee never chases the bot; matches the 2026 state of the art (GitHub Copilot code review auto-resolves addressed comments since 2026-09) | Wrong resolution hides an unfixed defect — needs an evidence gate |
| Leave resolution to humans (status quo) | No wrong resolutions | Threads rot; the reviewee must ping or resolve the reviewer's threads |
| Resolve on any new commit touching the file | Simple | Resolves on unrelated edits — worse than status quo |

## Decision

Auto-resolve with an evidence gate, own threads only:
1. Scope: only threads the reviewer itself opened. Never resolve another reviewer's thread.
2. Evidence gate: re-read the anchored span at the new head and the diff of the commits since the finding was posted. Resolve only when the posted fix, or an equivalent that removes the defect, is present. Ambiguous: leave open.
3. Order: reply first (one line naming the fixing commit), then resolve — the trail shows why the thread closed.
4. An applied suggestion resolves the thread natively on GitLab; the pass covers the manual-fix path and GitHub.
5. Unfixed threads stay open silently; no nag replies.

## Consequences

- The re-review pass in the loop gains a resolution step; the resolve commands already exist in both reference files (GitLab `resolved=true` PUT, GitHub `resolveReviewThread` mutation).
- The evidence gate reuses the read-first rule from [suggestion-assembly](./suggestion-assembly.md): no resolution without the real code open.
- A wrongly resolved thread remains visible in the MR/PR history with the naming reply — auditable, reversible by any participant.
