# GitLab Review

Reference file of the [`code-review`](SKILL.md) skill. Read it when the review lives on
a GitLab merge request: reading threads, posting replies or suggestions, resolving
threads, or troubleshooting `glab`.

## Overview

GitLab-specific tooling for the review flow. It provides the `glab` commands and API
calls; the flow itself is not here.

**REQUIRED BACKGROUND:** the [`code-review`](SKILL.md) skill defines the flow (read -> preview ->
approve -> post + resolve), the preview format, the Disposition rules, and the red
flags. Load it first. Git rules are in the [`git-conventions`](../git-conventions/SKILL.md) skill.

## Reading the review

Find the merge request. With no id, `glab` uses the current branch's MR.

```bash
glab mr list --reviewer=@me            # MRs waiting on me
glab mr list --author=@me              # my own MRs
glab mr view <iid> --unresolved        # human-readable open threads
```

Get structured discussions, which is what to work from:

```bash
# All unresolved diff comments with file, line, author, body and discussion id
glab mr note list <iid> -F json --state unresolved --type diff \
  --jq '.[] | {id, notes: [.notes[] | {author: .author.username, body,
        file: .position.new_path, line: .position.new_line,
        old_line: .position.old_line}]}'

glab mr note list <iid> -F json --state unresolved --type general   # non-diff comments
glab mr note list <iid> --file <path>                               # threads on one file
```

Keep the full `id` of each discussion: it is what `--reply` targets. Human-readable
output truncates it to the first eight characters, which `--reply` also accepts.

For diff context:

```bash
glab mr diff <iid>
glab mr view <iid> -F json --jq '.diff_refs'   # base_sha, head_sha, start_sha
```

In the preview, **File:line** is `new_path:new_line` and **Discussion** is the
eight-character id prefix.

## Suggestion syntax

GitLab applies a fenced `suggestion` block when the note is attached to the diff.
The range is relative to the commented line:

- `suggestion:-0+0` replaces the commented line only
- `suggestion:-1+2` replaces one line above through two lines below

Rules:
- The block content is the final code, with the file's real indentation, and no diff markers.
- Suggestions work on diff notes only. A general MR comment cannot carry an applicable suggestion.
- A suggestion edits only the anchored file. A fix in another file needs its own note
  anchored on a kept or added line of that file; no such line in the diff means prose.
- A reply inside a diff thread can carry a suggestion; it applies to that thread's line.
- Range cap: 201 changed lines per suggestion (100 above + 100 below the commented
  line). A wider span falls back to prose per the ladder.

### Insertion

A suggestion adds lines as well as replacing them. Anchor on the kept or added line
adjacent to the insertion point, use `suggestion:-0+0`, and make the block that line
verbatim (read from the file, exact indentation) followed by the new lines. Applying
keeps the anchored line and inserts the rest. A missing test posts this way: anchor on
the closing brace of the last test (or another kept line in the test file's diff),
never as a plain ```` ```csharp ```` block on the production file — that has no Apply
button.

## Posting a suggestion

The classification ladder in the [`code-review`](SKILL.md) skill decides *when*; this
is *how*. Anchor on `new_line` (right side) only — a suggestion anchored via
`old_line` is unreliable and never posted, which is the mechanical restatement of the
ladder's ban on deleted-line targets.

Fetch `diff_refs` immediately before posting — stale SHAs return 400:

```bash
glab mr view <iid> -F json --jq '.diff_refs'   # base_sha, head_sha, start_sha
```

Write the body to a file (heredoc, as above) with the `suggestion:-N+M` fence, then
create the diff discussion:

```bash
glab api --method POST "projects/<project-id>/merge_requests/<iid>/discussions" \
  -f "position[position_type]=text" \
  -f "position[base_sha]=<base_sha>" \
  -f "position[head_sha]=<head_sha>" \
  -f "position[start_sha]=<start_sha>" \
  -f "position[new_path]=src/Domain/Booking.cs" \
  -f "position[new_line]=42" \
  -f "body=$(cat body.md)"
```

Error paths:
- **400 on stale refs**: refetch `diff_refs` once, retry once, then post the finding
  as prose stating the concrete fix.
- **Rate cap**: gitlab.com caps note creation at 60/minute. Serialise posts; never
  create notes in parallel.

### Applying (reviewee side)

- UI: **Apply suggestion**, or add several to a batch — one commit for the batch.
- API: `PUT /suggestions/:id/apply` and `PUT /suggestions/batch_apply` (body `ids[]`),
  both with optional `commit_message`.
- A single apply credits the suggester as commit author; a batch apply credits the
  applier. Applying resolves the thread.

## Posting after approval

Two writes per approved `reply + resolve` thread, in this order. A thread is not done
after the reply.

**1. Reply** inside the reviewer's own thread, which is the default choice:

```bash
glab mr note create <iid> --reply <discussion-id> -m "$(cat body.md)"
```

Start a new inline thread when there is no existing discussion on that line:

```bash
glab mr note create <iid> --file src/Domain/Booking.cs --line 42 -m "$(cat body.md)"
glab mr note create <iid> --file src/Domain/Booking.cs --line 40:44 -m "$(cat body.md)"  # range
glab mr note create <iid> --file src/Domain/Booking.cs --old-line 42 -m "..."            # removed line
```

Write the body to a file first with a heredoc, because fenced blocks and backticks do
not survive inline shell quoting. `--file`, `--reply` and `--unique` are mutually
exclusive, so a reply cannot carry `--unique`: to stay idempotent on a retry, re-read
the thread and skip the ones that already have your note.

**2. Resolve** the thread. `glab` has no resolve verb, so go through the API — the
discussion id is the same one `--reply` took:

```bash
glab api --method PUT "projects/<project-id>/merge_requests/<iid>/discussions/<discussion-id>?resolved=true"
```

Get `<project-id>` once with `glab repo view -F json --jq .id`, or use the URL-encoded
path (`d-edge%2F...`). Requires the full discussion id, not the eight-character prefix.

**3. Verify** before reporting, as the [`code-review`](SKILL.md) skill requires:

```bash
glab mr note list <iid> -F json --jq '[.[] | {id, resolved: ([.notes[] | select(.system==false) | .resolved] | any)}]'
```

## Setup

Check auth with `glab auth status`. On failure, tell the user to run
`glab auth login --stdin` with your GitLab host, for example `--hostname gitlab.com`,
and a personal access token scoped `api`; do not attempt to create or read tokens.
