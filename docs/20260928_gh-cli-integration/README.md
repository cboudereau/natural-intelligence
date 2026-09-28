# gh-cli-integration

GitHub CLI (`gh`) support mirroring the glab integration, plus the git-versus-forge rules that stop agents mixing `git` and forge commands. Integrated 2026-09-28, plugin version 1.4.0.

## Design

- [gh-cli-integration](designs/gh-cli-integration.md) — requirements, boundary, failure modes, rollout

## Decisions (ADRs, accepted)

- [Which GitHub CLI](adrs/github-cli-choice.md) — `gh`, with the bias sweep of the evidence
- [Platform detection](adrs/platform-detection.md) — argument override, then origin host, then auth hosts, then ask; never guess
- [Git-versus-forge boundary](adrs/forge-first-boundary.md) — forge CLI owns server-side operations; `git` owns local state only
- [Platform reference files](adrs/platform-reference-files.md) — commands stay platform-agnostic; per-platform `.md` files carry the commands

## What changed in the plugin

- [`skills/git-conventions/SKILL.md`](../../skills/git-conventions/SKILL.md): platform routing rule, git-versus-forge boundary table, forge-based MR/PR description flow (clip.exe removed), commit type `docs:`
- [`skills/git-conventions/gitlab.md`](../../skills/git-conventions/gitlab.md) and [`github.md`](../../skills/git-conventions/github.md): MR/PR lifecycle commands per platform
- [`skills/code-review/github.md`](../../skills/code-review/github.md): GitHub review threads (GraphQL) mirroring [`gitlab.md`](../../skills/code-review/gitlab.md)
- [`commands/review-loop.md`](../../commands/review-loop.md) and [`commands/merge-loop.md`](../../commands/merge-loop.md): platform-agnostic, no forge CLI named
- [`skills/plan/SKILL.md`](../../skills/plan/SKILL.md): Phase 4c pre-flight gains the link lint

Follow-up: [plan-skill-split](../workspace/plan-skill-split/DESIGN.md) (seeded, scope fixed).
