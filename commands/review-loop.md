---
description: Review MRs assigned to me in a loop, post findings, report the links
argument-hint: "[project path or URL]"
---
Start a dynamic /loop that reviews every change (MR/PR) awaiting my review in the project
given in $ARGUMENTS (default: the current repository's origin). Resolve the platform first,
per the routing rule in `${CLAUDE_PLUGIN_ROOT}/skills/git-conventions/SKILL.md#platform-routing`.
The `ni:code-review` skill owns the review itself (built-in pass, checklist subagents,
finding format); this loop owns the mechanics around it. Platform commands are never
named here: review threads and listing live in
`${CLAUDE_PLUGIN_ROOT}/skills/code-review/gitlab.md` or `github.md`; MR/PR lifecycle
(approvals, diff) in `${CLAUDE_PLUGIN_ROOT}/skills/git-conventions/gitlab.md` or
`github.md`.

Each iteration:

1. List open changes where I am reviewer: listing commands in the code-review
   reference file for the routed platform.
2. Skip a change I already approved (approval state: lifecycle reference file), or one
   reviewed in an earlier iteration that has no new commits and no reviewer replies.
3. Review each remaining change with the giving-a-review flow of `ni:code-review`.
4. Diff via the forge, per the git-versus-forge boundary in
   `${CLAUDE_PLUGIN_ROOT}/skills/git-conventions/SKILL.md`: the forge diff is against
   the merge-base of the source branch and its target, never two-dot against the
   target head — a branch forked before later merges shows those merges as deletions.
5. Post each finding as its own discussion on the change, severity-prefixed, with
   `file:line` in the body: posting commands in the code-review reference file. This
   command's standing instruction is the approval for these first-review comments.
6. Report the posted comment links grouped by change: one section per change, links
   listed under it.
7. When a loop finding conflicts with an existing comment or thread, do not publish
   that finding: hold it, ask me for help with both positions summarised, and post
   only what I decide. The standing approval never covers a conflicting comment.
8. Carry state forward in the loop prompt: append the reviewed change ids with
   "skip unless new commits or reviewer replies".

Loop mechanics: run the check now, then ScheduleWakeup with the amended prompt. Idle tick
1200-1800 s; review subagents notify on completion, so the wakeup is only a fallback. Any
prompt amendment (skip rules, output grouping) rewrites the ScheduleWakeup prompt, never a
separate note.

Boundaries: never approve, merge, close, or resolve anything: those stay my calls. Replies
to existing reviewer threads keep the preview-then-approval flow of `ni:code-review`; only
first-review comments ride the standing approval. The loop dies with the session; a durable
schedule is /schedule, not /loop.
