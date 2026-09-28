---
status: accepted
---
# Platform detection

Addresses: [FR1](../designs/gh-cli-integration.md#fr1), [FR3](../designs/gh-cli-integration.md#fr3)

## Problem

No rule tells an agent whether a repository lives on GitLab or GitHub, so nothing routes between `glab` and `gh`. Today the only implicit detection is `glab`'s current-branch MR lookup ([`code-review/gitlab.md:18`](../../../skills/code-review/gitlab.md)), which fails silently on a GitHub repository.

## Options

| Option | Pros | Cons |
|---|---|---|
| Origin host mapping | One `git remote get-url origin` call; deterministic; works offline; explicit override stays possible | Needs a fallback for unknown hosts (GHES, self-hosted GitLab) |
| Try both CLIs, keep the one that answers | No mapping table | Two network calls; auth errors are indistinguishable from wrong-platform errors; slow and noisy |
| Always ask the user | Never wrong | Interrupts every loop run; the answer is already in the remote URL |

## Decision

Origin host mapping, owned by [`git-conventions`](../../../skills/git-conventions/SKILL.md):
1. An explicit argument or user statement wins (a project path or URL passed to a loop command names the platform).
2. Otherwise read `git remote get-url origin`. Host `github.com` routes to `gh`; host containing `gitlab` routes to `glab`.
3. Unknown host: check `gh auth status` and `glab auth status` for a matching configured host; if still ambiguous, ask the user. Never guess.

## Consequences

- [`code-review/SKILL.md`](../../../skills/code-review/SKILL.md) replaces "for GitLab, read gitlab.md" with one routing line pointing at this rule, then to [`gitlab.md`](../../../skills/code-review/gitlab.md) or [`github.md`](../../../skills/code-review/github.md).
- The loop commands drop "gitlab" from their argument hints and gain no platform flag: routing is automatic.
- Self-hosted instances with non-standard hostnames cost one question to the user, once per repository.
