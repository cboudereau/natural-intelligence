# GitHub PR Lifecycle

Reference file of the [`git-conventions`](SKILL.md) skill. Read it when a PR lifecycle
operation targets GitHub: create, describe, diff, gate, merge, rebase, CI status, auth.

**REQUIRED BACKGROUND:** the [`git-conventions`](SKILL.md) skill owns the platform
routing rule and the git-versus-forge boundary. Load it first. Review threads and
suggestions belong to the [`code-review`](../code-review/SKILL.md) skill.

`gh` syntax below is from documented behaviour (`manual: no gh binary in this
environment`).

## Create and describe

Write the description markdown to a file, then:

```bash
gh pr create --title "<title>" --body-file <file>
```

Edit an existing description:

```bash
gh pr edit <number> --body-file <file>
```

## Diff

```bash
gh pr diff <number>
```

Local `git diff` covers uncommitted work only.

## Merge gates

Merge only when GitHub reports the PR clean, approved, and not a draft:

```bash
gh pr view <number> --json mergeStateStatus,reviewDecision,isDraft
```

Gate: `mergeStateStatus == "CLEAN"`, `reviewDecision == "APPROVED"`, and
`isDraft == false`. Never bypass a failing gate.

## Merge, auto-merge, cancel

```bash
gh pr merge <number>                   # merge now
gh pr merge <number> --auto            # merge when requirements pass
gh pr merge <number> --disable-auto    # cancel auto-merge
```

## Rebase / branch update

Forge only, never local rebase plus force-push:

```bash
gh api repos/{owner}/{repo}/pulls/{number}/update-branch -X PUT
```

## CI status

```bash
gh pr checks <number>
```

## Auth

```bash
gh auth status
```

On failure, tell the user to run `gh auth login`. Never create, read, or store
tokens yourself.
