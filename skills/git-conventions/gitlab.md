# GitLab MR Lifecycle

Reference file of the [`git-conventions`](SKILL.md) skill. Read it when an MR lifecycle
operation targets GitLab: create, describe, diff, gate, merge, rebase, CI status, auth.

**REQUIRED BACKGROUND:** the [`git-conventions`](SKILL.md) skill owns the forge
routing rule and the git-versus-forge boundary. Load it first. Review threads and
suggestions belong to the [`code-review`](../code-review/SKILL.md) skill.

## Create and describe

Write the description markdown to a file with the Write tool, pick the reviewer (rule
below), then:

```bash
glab mr create --title "<title>" --description-file <file> --assignee @me --reviewer <username>
```

Commands follow the shell rules in [`software-engineer`](../software-engineer/SKILL.md#onboarding).

Reviewer selection, in order:
1. `CODEOWNERS` exists (root, `.gitlab/` or `docs/`): the owners matching the touched paths.
2. Otherwise the top recent committer of the touched files, excluding the author and bots:
   `git shortlog -sne --since="6 months ago" -- <paths>`
3. No candidate: create without `--reviewer` and tell the user to pick one.

The assignee is always me (`--assignee @me`).

Edit an existing description. Write the description to a file with the Write tool, then:

```bash
glab mr update <iid> --description-file <file>
```

## Diff

```bash
glab mr diff <iid>
```

Local `git diff` covers uncommitted work only.

## Review verdict

Submit the verdict of the [`code-review`](../code-review/SKILL.md#verdict) skill.
Write the summary to `verdict.md` with the Write tool first.

```bash
glab mr approve <iid>
glab mr revoke <iid>
glab api --method POST "projects/:id/merge_requests/<iid>/notes" -F body=@verdict.md
```

`approve` runs the first line. `changes required` posts the summary note, and runs
`revoke` first when you approved an earlier head. GitLab has no request-changes state
in `glab`, so the note carries the verdict.

## Merge gates

Merge only when GitLab reports the MR mergeable and not a draft:

```bash
glab api "projects/:id/merge_requests/<iid>" --jq '.detailed_merge_status, .draft, [.reviewers[].username]'
glab api "projects/:id/merge_requests/<iid>/approvals" --jq '.approved, [.approved_by[].user.username]'
```

Gate: `detailed_merge_status == "mergeable"`, `draft == false`, `approved == true`,
and `approved_by` not empty (at least one approval). Never bypass a failing gate.

## Merge, auto-merge, cancel

Re-read the head SHA before each merge, and always pass it. Some projects require it
(400 "SHA must be provided"), and it makes the merge fail if the head moved.

```bash
glab api "projects/:id/merge_requests/<iid>" --jq '.sha'
glab mr merge <iid> --sha <sha>                    # merge now
glab mr merge <iid> --sha <sha> --auto-merge       # merge when pipeline succeeds
# cancel auto-merge
glab api -X POST "projects/:id/merge_requests/<iid>/cancel_merge_when_pipeline_succeeds"
```

### Stacked MRs

Top-down: the child merges first, into its parent's branch. Bottom-up: the parent
merges first, into the target.

- Top-down resets approvals: when the project resets approvals on push, merging a
  child into its parent resets every parent's approval.
- Bottom-up with squash conflicts: the child branch still holds the parent's original
  commits, so its squash fails with "Squashing failed: The commits have conflicting
  changes. Squash the commits locally, resolve the conflicts, then push the branch."
  A merge without squash then fails too, with "This merge request has merge conflicts".
- Fix: merge the target branch into the child and push without force, or fold the
  child into the next MR.

After a squash merge, or a merge of the target into a branch, the commits no longer show
whether content reached the target. Prove it by comparing trees: `git diff` prints
nothing, or the compare with `straight=true` returns no diffs. Never rely on ahead or
behind counts, or on the default compare, which diffs from the merge base.

```bash
git diff <target> <branch>
glab api "projects/:id/repository/compare?from=<target>&to=<branch>&straight=true" --jq ".diffs | length"
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
