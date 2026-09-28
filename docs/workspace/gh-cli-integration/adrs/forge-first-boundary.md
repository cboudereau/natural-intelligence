---
status: accepted
---
# Git-versus-forge boundary

Addresses: [FR4](../DESIGN.md#fr4), [FR3](../DESIGN.md#fr3)

## Problem

Agents mix `git` and forge commands because no rule assigns operations to tools. Found in the codebase: the MR/PR description flow ends at a clipboard (`git-conventions/SKILL.md:99`, `clip.exe`, Windows-only) with no create step; rebase is reachable server-side (`merge-loop.md:27-28`) and locally, where it collides with the no-force rule (`git-conventions/SKILL.md:29`); review diffs have two competing sources (`review-loop.md:16` vs `gitlab.md:45`). The user's requirement: prefer the forge CLI (`gh`/`glab`) over raw `git` wherever the operation touches the forge.

## Options

| Option | Pros | Cons |
|---|---|---|
| Forge-first: forge CLI owns every server-side operation, `git` owns only local state | One-line test for any operation ("does it live on the server?"); removes the rebase/no-force collision; removes the clipboard step; forge CLIs handle auth | Requires `gh`/`glab` installed and authenticated |
| Status quo: git everywhere, forge CLI ad hoc | Nothing to change | The six documented mixing points persist |
| Forge-only, including local operations | Maximal consistency | Impossible: commit, stage, worktree, and local diff have no forge equivalent; `hub`-style git proxying was rejected in [github-cli-choice](./github-cli-choice.md) |

## Decision

Forge-first. The rule, owned by `git-conventions`:

| Operation | Tool |
|---|---|
| stage, commit, branch, local diff, log, worktree | `git` only |
| push | `git push` (unchanged rules: on request, current branch, never force) |
| MR/PR create, description, edit | `glab mr create/update` / `gh pr create/edit` — never the clipboard |
| MR/PR diff for review | `glab mr diff` / `gh pr diff` — local `git diff` only for uncommitted work |
| threads, approvals, merge, CI status | forge CLI only |
| rebase / branch update of an MR/PR | forge only (`glab mr rebase`, `gh api .../update-branch`); never a local rebase plus force-push |

## Consequences

- The no-force rule stops colliding with rebase: local rebase of a pushed branch is now explicitly out.
- `clip.exe` disappears; the description flow works on Linux.
- The push rule keeps one owner (`git-conventions`); `code-review/SKILL.md:178` and `gitlab.md:13-14` become pointers.
- Agents whose tool list allows only `git` (e.g. `ni:reviewer` at `agents/reviewer.md:44`) need their diff instruction pointed at a fetched local ref or a forge diff passed in by the orchestrator — noted in the FR3/FR2 tasks.
