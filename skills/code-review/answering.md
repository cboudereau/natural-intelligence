# Answering review feedback — preview and report

Reference file of the [`code-review`](SKILL.md) skill. Read it when the answering flow
reaches the preview: the table formats, the dispositions, the markdown output, and the
report after approval.

## Preview format

One row per thread, two tables. **Replies** to existing reviewer threads (main case):

```markdown
| # | File:line | Discussion | Reviewer comment (verbatim) | Reply | Suggestion | Resolve? |
|---|---|---|---|---|---|---|
| 1 | src/Domain/Booking.cs:42 | abc12345 | "Verbatim reviewer comment." | Agreed, null check added | line: `if (booking is null) return NotFound();` | yes |
| 2 | src/Api/Handler.cs:17 | def67890 | "Verbatim reviewer comment." | Fixed, test still to add | range: one-line summary | no - test missing |
| 3 | general | 0123abcd | "Verbatim reviewer comment." | Question back to reviewer | none | no - needs @user |
```

**New comments** on lines with no existing thread:

```markdown
| # | File:line | Comment | Suggestion |
|---|---|---|---|
| 4 | src/Domain/Booking.cs:58 | Same null check as thread 1 applies here | line: one-line summary |
```

End with **Not addressed:** item, reason.

Column rules:
- **File:line** is the path and line on the new side of the diff, or `general` for a non-diff thread.
- **Discussion** is a short id prefix; keep the full id for the write.
- **Reviewer comment** is quoted verbatim, trimmed with `...` only when long.
- **Suggestion** is `line`, `range`, or `none`, plus the replacement when it fits one
  line, otherwise a summary; the full fenced block goes in the markdown file or the
  note body. The range syntax is forge-specific: see the forge file linked from [`code-review`](SKILL.md).
- **Resolve?** is `yes` for `reply + resolve`, or `no - reason` for `reply only` and
  `leave open`. It maps to the Disposition below.

Every thread in the preview carries a **Disposition**, whatever the output format.
It is a required field with exactly one of three values:

| Disposition | Meaning | Write on approval |
|---|---|---|
| `reply + resolve` | The comment is answered and the ask is done | reply, then resolve |
| `reply only` | Answered, but something real is still outstanding — say what | reply, leave open |
| `leave open` | Needs the user, another person, or a decision — say who or what | nothing |

`reply + resolve` is the normal case for a comment whose ask has landed; defaulting to
`reply only` to stay safe leaves the user closing threads by hand — the work this flow removes.

When a markdown output is asked, create one section per unresolved thread in
`<scratchpad>/<change-id>-suggestions.md`. The change-id is the MR iid or PR number:

````markdown
## 1. src/Domain/Booking.cs:42 - @reviewer  [discussion: abc12345]

> Verbatim reviewer comment.

**Assessment:** what the comment is actually asking, and whether it holds.

**Disposition:** reply + resolve

**Current code** (src/Domain/Booking.cs:40-44):
```csharp
<actual lines read from the file>
```

**Proposed reply** (the fence's range syntax is forge-specific):
```suggestion
<replacement for line 42>
```
````

End the file with a short **Not addressed** list for anything skipped, with the reason.

Then ask one question: which numbered items to write, taking the previewed dispositions
as read. Write only those, and resolve exactly the approved `reply + resolve` ones.
If the user narrows or overrides a disposition in their answer, theirs wins.

## After approval

Post the replies, resolve the approved threads, then verify and report in one message:
posted note ids, threads now resolved, and threads left open with their disposition's
reason. Commands: the forge file linked from [`code-review`](SKILL.md), per the [forge routing rule](../git-conventions/SKILL.md#forge-routing).
