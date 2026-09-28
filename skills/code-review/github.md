# GitHub Review

Reference file of the [`code-review`](SKILL.md) skill. Read it when the review lives on
a GitHub pull request: reading threads, posting replies or suggestions, resolving
threads, or troubleshooting `gh`.

## Overview

GitHub-specific tooling for the review flow. It provides the `gh` commands and API
calls; the flow itself is not here. Scope: review threads only. PR lifecycle (create,
merge, CI) belongs to the [`git-conventions`](../git-conventions/SKILL.md) skill.

**REQUIRED BACKGROUND:** the [`code-review`](SKILL.md) skill defines the flow (read ->
preview -> approve -> post + resolve), the preview format, the Disposition rules, and
the red flags. Load it first. Git rules are in the
[`git-conventions`](../git-conventions/SKILL.md) skill.

## Reading the review

Find the pull request. With no number, `gh` uses the current branch's PR.

```bash
gh pr list --search "review-requested:@me"   # PRs waiting on me
gh pr list --author "@me"                    # my own PRs
```

`gh` has no CLI verb for open review threads — no `--unresolved` equivalent. Go
through GraphQL for both the human-readable pass and the structured data:

```bash
# All review threads with resolution state, file, line, and comment ids
gh api graphql -f query='
  query($owner: String!, $repo: String!, $pr: Int!) {
    repository(owner: $owner, name: $repo) {
      pullRequest(number: $pr) {
        reviewThreads(first: 100) {
          nodes {
            id isResolved path line
            comments(first: 50) {
              nodes { databaseId author { login } body }
            }
          }
        }
      }
    }
  }' -f owner=<owner> -f repo=<repo> -F pr=<number>
```

Get `<owner>` and `<repo>` once with `gh repo view --json owner,name`. Filter to
open threads with `--jq '... | select(.isResolved | not)'` on the nodes.

Keep two ids per thread: the GraphQL thread `id` (what resolve takes) and the first
comment's `databaseId` (what the REST reply endpoint takes).

For diff context:

```bash
gh pr diff <number>
gh pr view <number> --json baseRefOid,headRefOid   # base and head refs
```

In the preview, **File:line** is the thread's `path:line` and **Discussion** is a
short prefix of the thread id.

## Suggestion syntax

GitHub applies a fenced `suggestion` block when the comment is attached to the diff.
Unlike GitLab, there is **no `:-N+M` range modifier**: the block header is bare
` ```suggestion `. The suggestion replaces the line span the comment anchors —
one line, or the `start_line`..`line` range set when the comment was created.
A wider replacement needs a new comment anchored on the wider range.

Rules:
- The block content is the final code, with the file's real indentation, and no diff markers.
- Suggestions work on diff comments only. A general PR comment cannot carry an applicable suggestion.
- A reply inside a diff thread can carry a suggestion; it applies to that thread's anchored span.

## Posting after approval

Two writes per approved `reply + resolve` thread, in this order. A thread is not done
after the reply.

**1. Reply** inside the reviewer's own thread, which is the default choice. The REST
replies endpoint takes the first comment's `databaseId`:

```bash
gh api repos/{owner}/{repo}/pulls/{number}/comments/{comment_id}/replies \
  -f body=@body.md
```

Write the body to a file first with a heredoc, because fenced blocks and backticks do
not survive inline shell quoting. There is no uniqueness flag: to stay idempotent on a
retry, re-read the thread and skip the ones that already have your note.

Start a new inline thread when there is no existing discussion on that line:

```bash
gh api repos/{owner}/{repo}/pulls/{number}/comments \
  -f body=@body.md -f commit_id=<headRefOid> -f path=src/Domain/Booking.cs \
  -F line=42 -f side=RIGHT
# range: add -F start_line=40 -f start_side=RIGHT
# removed line: -f side=LEFT with the old line number
```

**2. Resolve** the thread. Resolution is GraphQL-only — the thread id is the one from
the reading query, not a comment id:

```bash
gh api graphql -f query='
  mutation($id: ID!) {
    resolveReviewThread(input: {threadId: $id}) {
      thread { id isResolved }
    }
  }' -f id=<thread-id>
```

If the mutation fails, report that thread id and the error, then continue with the
next thread. Never stop the pass on one failed resolve.

**3. Verify** before reporting, as the [`code-review`](SKILL.md) skill requires:
re-run the reading query and check `isResolved` on every thread you touched.

## Setup

Check auth with `gh auth status`. On failure, tell the user to run `gh auth login`;
do not attempt to create or read tokens.

`gh` syntax in this file is from documented behaviour (`manual: no gh binary in this
environment`); flag and field spellings were not proven by a live call.
