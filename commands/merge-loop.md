---
description: Merge my approved MRs or PRs in a loop, report what merged and what is blocked
argument-hint: "[project path or URL]"
---
Start a dynamic /loop that merges every approved change (MR/PR) authored by me in the
project given in $ARGUMENTS (default: the current repository's origin). Resolve the
forge first, per the forge routing rule of the `ni:git-conventions` skill.
Forge commands are never named here: MR/PR lifecycle (state, gates, merge,
auto-merge, cancel, branch update, CI status) lives in the gitlab.md or github.md
reference file of `ni:git-conventions`; listing lives in the gitlab.md or github.md
reference file of `ni:code-review`. This command's
standing instruction is the approval for each merge that passes every gate below;
anything short of all gates is reported, never merged.

Each iteration:

1. List my open changes: own-changes listing command in the code-review reference
   file for the routed forge.
2. For each change, read its state from the forge: merge state, draft flag, and
   approvals (commands: lifecycle reference file).
3. Merge gates, all required: merge only when the forge reports the change
   mergeable, approved, and not a draft, with at least one approval, and no
   unresolved threads. Never bypass a failing gate.
4. Gates pass and CI succeeded: merge. Gates pass but CI still running: set
   auto-merge, then re-check next iteration. If an approval is revoked afterwards,
   cancel it (merge, auto-merge, and cancel commands: lifecycle reference file).
5. The forge reports the change behind its target: update the branch via the forge
   only, never a local rebase, per the git-versus-forge boundary of the
   `ni:git-conventions` skill (update command: lifecycle reference file). Re-check
   next iteration.
6. Failed CI, conflicts, missing approval, or unresolved threads: skip, and report
   the change with its blocking reason. For a missing approval, name the reviewers
   who have not approved yet.
7. Report each iteration: merged changes with links, auto-merge set, blocked changes
   with reasons.
8. Carry state forward in the loop prompt: append merged change ids as done, and
   pending ids with their last known blocker.
9. Stop the loop when no open change authored by me remains.

Loop mechanics: run the check now, then ScheduleWakeup with the amended prompt. Waiting
on CI: match the delay to its usual duration (300-600 s). Otherwise idle tick
1200-1800 s. Any prompt amendment rewrites the ScheduleWakeup prompt, never a separate
note.

Task panel: the one-line loop status below the input box comes from the ScheduleWakeup
`reason`. Make it the iteration status, not a generic wait: counts plus the change
waited on, for example "1 merged, auto-merge on !18; !21 blocked (reviewer approval)".
Set `noop: false` on a tick that merged, set auto-merge, or found a new blocker,
`noop: true` on a no-change tick so quiet ticks collapse in the panel.

Boundaries: never approve my own changes, never merge with a failed or absent CI run,
never force merge, never resolve someone else's thread, never touch changes I did not
author. Fixing a failed CI run or answering threads is separate work: notify me instead.
The loop dies with the session; a durable schedule is /schedule, not /loop.
