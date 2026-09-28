---
status: accepted
---
# Platform reference files

Addresses: [FR2](../designs/gh-cli-integration.md#fr2), [FR3](../designs/gh-cli-integration.md#fr3), [FR4](../designs/gh-cli-integration.md#fr4)

## Problem

Where do platform-specific commands live? The first draft of FR3 paired every `glab` step with its `gh` twin inline in the loop commands. That doubles every step, and each new forge (Bitbucket, Gitea, ...) would touch every command file again.

## Options

| Option | Pros | Cons |
|---|---|---|
| Inline command pairs in each command file | Self-contained files | Every step duplicated per platform; adding a forge edits every command; violates one rule, one owner |
| Platform-agnostic flows + per-platform reference `.md` files | Commands describe the flow once; adding a forge adds one reference file per owning skill; matches the existing [`code-review/gitlab.md`](../../../skills/code-review/gitlab.md) pattern | One indirection when executing |

## Decision

Platform-agnostic flows with per-platform reference files (user decision, 2026-09-28). Rules:

1. `commands/*.md` name no forge CLI. They describe the flow (list, gate, merge, rebase) in platform-neutral terms and route via the [`git-conventions`](../../../skills/git-conventions/SKILL.md) platform routing rule to the right reference file.
2. Each skill that owns a domain holds one reference file per platform: [`code-review/gitlab.md`](../../../skills/code-review/gitlab.md) and [`code-review/github.md`](../../../skills/code-review/github.md) for review threads; [`git-conventions/gitlab.md`](../../../skills/git-conventions/gitlab.md) and [`git-conventions/github.md`](../../../skills/git-conventions/github.md) for the MR/PR lifecycle (create, description, diff, merge gates, merge, auto-merge, rebase or branch update, CI status, auth).
3. [`git-conventions/SKILL.md`](../../../skills/git-conventions/SKILL.md) keeps the boundary table at tool-name level (`git` vs forge CLI) and links the reference files for concrete commands.
4. A new forge means new reference files, no command edits.

## Consequences

- Loop commands shrink and stop naming `glab`; the GitLab-only fields (`detailed_merge_status`, cancel API) move into [`git-conventions/gitlab.md`](../../../skills/git-conventions/gitlab.md); the GitHub equivalents (`mergeStateStatus`, `reviewDecision`, `update-branch`) go into [`git-conventions/github.md`](../../../skills/git-conventions/github.md).
- Two reference files named [`gitlab.md`](../../../skills/code-review/gitlab.md)/[`github.md`](../../../skills/code-review/github.md) exist per owning skill; links always carry the skill path, so no ambiguity.
- The merge-gate semantics stay per-platform where they belong; commands state only the neutral rule (merge only when the platform reports the change mergeable and approved; never bypass a gate).
