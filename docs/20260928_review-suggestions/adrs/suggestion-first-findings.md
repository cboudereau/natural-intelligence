---
status: accepted
---
# Suggestion-first findings

Addresses: [FR1](../designs/review-suggestions.md#fr1), [FR4](../designs/review-suggestions.md#fr4)

## Problem

Findings post as prose comments; the reviewee retypes every fix by hand. Both platforms offer one-click application of suggestion blocks, but only when the comment is diff-anchored and the target lines qualify. When does a finding become a suggestion, and what happens when it cannot?

## Options

| Option | Pros | Cons |
|---|---|---|
| Suggestion-first with a fallback ladder | Reviewee one-clicks the easy fixes; consensus faster; prose stays for judgement calls | Classification rule to maintain; anchoring failures need handling |
| Always prose (status quo) | Nothing to change | Every fix retyped by the reviewee |
| Always suggestion | Uniform | Impossible: deleted lines fail on GitHub, non-diff notes render dead fences, design findings have no replacement text; noisy wrong suggestions burn reviewer trust |

## Decision

Suggestion-first with this ladder. A finding posts as a suggestion when ALL hold:
1. The fix is mechanical: replacement text for one contiguous span, no judgement left to the reviewee.
2. The span anchors on kept or added lines (`new_line` / `side=RIGHT`) inside the current diff. Deleted-line targets never carry a fence (GitHub hard-fails; GitLab unreliable — gitlab-org/gitlab#293642).
3. The span fits one hunk and, on GitLab, 201 lines.
Otherwise the finding posts as a prose comment that still states the concrete fix. On anchoring errors (stale refs, outdated comment): refetch once, retry once, then prose.

Severity is orthogonal: a `c:` (critical) finding with a mechanical fix still gets a suggestion — the block eases the fix, the severity drives the disposition.

## Evidence and bias sweep

Sources are official docs [D] and community reports [C], tagged in the research: GitLab suggestion syntax, cap and apply API ([docs.gitlab.com/user/project/merge_requests/reviews/suggestions](https://docs.gitlab.com/user/project/merge_requests/reviews/suggestions/), [docs.gitlab.com/api/suggestions](https://docs.gitlab.com/api/suggestions/)); GitHub anchoring and apply-is-UI-only ([docs.github.com/en/rest/pulls/comments](https://docs.github.com/en/rest/pulls/comments), [GraphQL mutations reference](https://docs.github.com/en/graphql/reference/mutations) — no apply mutation, 2026); deleted-line failures ([github community #32114](https://github.com/orgs/community/discussions/32114)).

| Bias | Mechanism | Direction | Evidence | Magnitude | Verdict |
|---|---|---|---|---|---|
| Publication / vendor | Platform docs undersell their gaps | Inflates both platforms | The two load-bearing negatives (GitHub no apply API, deleted-line failures) come from community reports and an absence check, not vendor claims | Weak | Rejected — negatives independently sourced |
| Availability | Vivid community failure threads could overweight edge cases | Deflates "always suggestion" | Deleted-line failure confirmed on both platforms across years of reports | Weak | Rejected — the ladder costs nothing when the edge case is absent |
| Recency | 2025-2026 previews (GitHub unchanged-line commenting) could look production-ready | Inflates GitHub | Changelog itself says API creation "currently limited" | Moderate | Confirmed — preview excluded from the design (non-goal) |

## Consequences

- Reviewees apply mechanical fixes with one click; on GitLab a batch becomes one commit.
- The classification rule lives once, in the code-review skill; the loop and reference files point to it.
- Prose fallback keeps every finding postable — no finding is dropped because it cannot anchor.
- Applied suggestions credit the bot as author (GitLab single apply) or co-author (GitHub) — teams with "committers cannot approve" rules see the bot in the trailer, which is honest attribution.
