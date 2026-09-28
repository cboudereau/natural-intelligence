# GitLab MR Lifecycle

Reference file of the [`git-conventions`](SKILL.md) skill. Read it when an MR lifecycle
operation targets GitLab: create, describe, diff, gate, merge, rebase, CI status, auth.

**REQUIRED BACKGROUND:** the [`git-conventions`](SKILL.md) skill owns the platform
routing rule and the git-versus-forge boundary. Load it first. Review threads and
suggestions live in [code-review/gitlab.md](../code-review/gitlab.md).

## Create and describe

Write the description markdown to a file, then:

```bash
glab mr create --title "<title>" --description-file <file>
```

Edit an existing description:

```bash
glab mr update <iid> --description "$(cat <file>)"
```

## Diff

```bash
glab mr diff <iid>
```

Local `git diff` covers uncommitted work only.

## Merge gates

Merge only when GitLab reports the MR mergeable and not a draft:

```bash
glab api "projects/:id/merge_requests/<iid>" --jq '.detailed_merge_status, .draft, [.reviewers[].username]'
glab api "projects/:id/merge_requests/<iid>/approvals" --jq '.approved, [.approved_by[].user.username]'
```

Gate: `detailed_merge_status == "mergeable"`, `draft == false`, `approved == true`,
and every reviewer username present in `approved_by`. `approved` alone only means the
approval rules are met. Never bypass a failing gate.

## Merge, auto-merge, cancel

```bash
glab mr merge <iid>                    # merge now
glab mr merge <iid> --auto-merge       # merge when pipeline succeeds
# cancel auto-merge
glab api -X POST "projects/:id/merge_requests/<iid>/cancel_merge_when_pipeline_succeeds"
```

## Rebase / branch update

Forge only, never local rebase plus force-push:

```bash
glab mr rebase <iid>
```

## CI status

```bash
glab ci status
glab ci status --branch <branch>
```

## Auth

```bash
glab auth status
```

On failure, tell the user to run `glab auth login`. Never create, read, or store
tokens yourself.
