# review-suggestions

Review findings now post as one-click applicable suggestions on GitLab and GitHub, and the reviewer resolves its own threads once a new commit fixes the finding. Integrated 2026-09-28, plugin version 1.5.0. Amends [20260928_gh-cli-integration](../20260928_gh-cli-integration/README.md).

## Design

- [review-suggestions](designs/review-suggestions.md) — requirements, 2026 state of the art, failure modes

## Decisions (ADRs, accepted)

- [Suggestion-first findings](adrs/suggestion-first-findings.md) — suggestion when mechanical + kept/added diff lines + one hunk; prose fallback always states the fix
- [Suggestion assembly](adrs/suggestion-assembly.md) — the poster re-reads the real file and builds the block as an edited copy
- [Thread auto-resolution](adrs/thread-auto-resolution.md) — own threads close after an evidence check, reply names the fixing commit

## What changed in the plugin

- [`skills/code-review/SKILL.md`](../../skills/code-review/SKILL.md): classification ladder, generalised read-first rule, posting step in the giving-a-review flow, own-thread resolution rule, GitLab-only syntax removed
- [`skills/code-review/gitlab.md`](../../skills/code-review/gitlab.md): suggestion posting payload (fresh `diff_refs`, `new_line` anchoring, 201-line cap, rate and stale-ref error paths, apply API facts)
- [`skills/code-review/github.md`](../../skills/code-review/github.md): batched review call with anchored `comments[]`, bare fence, deleted-line and file-level limits, UI-only apply, reading query gains `startLine`/`diffSide`
- [`commands/review-loop.md`](../../commands/review-loop.md): findings anchor on the diff with suggestions per the ladder; re-review step auto-resolves own fixed threads; keep-it-simple note
