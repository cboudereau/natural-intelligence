# review-suggestions — Design Doc

Amends: [20260928_gh-cli-integration](../../20260928_gh-cli-integration/README.md) (extends the platform reference files and the review loop it created).

## Context

The review flow posts findings as prose comments. Suggestion blocks — which give the reviewee a one-click "Apply suggestion" / "Commit suggestion" button and ease consensus — exist only in the answering flow of [`code-review/SKILL.md`](../../../skills/code-review/SKILL.md) (the reviewee replying on their own change). Investigation found nine gaps; the load-bearing ones:

- No reviewer-side rule: neither the giving-a-review flow ([`code-review/SKILL.md:33-54`](../../../skills/code-review/SKILL.md)) nor [`review-loop`](../../../commands/review-loop.md) step 5 attaches a fix as a suggestion.
- The finding format ([`code-review/SKILL.md:60`](../../../skills/code-review/SKILL.md), [`agents/reviewer.md:25-30`](../../../agents/reviewer.md)) carries a prose fix — no replacement code, span, or indentation.
- Findings post as general discussions, which cannot carry an applicable suggestion on either platform.
- The platform-agnostic [`code-review/SKILL.md:253-256`](../../../skills/code-review/SKILL.md) template hard-codes GitLab `suggestion:-0+0` syntax; on GitHub that posts dead markdown.
- Removed-line and multi-line anchoring rules are missing from both reference files.

State of the art (2026, sources in [ADR: suggestion-first-findings](../adrs/suggestion-first-findings.md)): GitLab applies suggestions per note or batch, with a documented apply API; GitHub applies only through the UI (no API, confirmed absent 2026); both need exact diff anchoring (GitLab `position` + `suggestion:-N+M`, cap 201 lines; GitHub `start_line`/`line` + bare fence); suggestions on deleted lines fail on GitHub and are unreliable on GitLab; GitLab caps notes at 60/minute; GitHub wants all inline comments batched in one review call.

## Functional Requirements

### <a id="fr1"></a>FR1 — Fix classification rule
[`code-review/SKILL.md`](../../../skills/code-review/SKILL.md) gains one reviewer-side rule deciding suggestion versus prose. A finding posts as a suggestion when the fix is mechanical (replacement text for a contiguous span), the span anchors on kept-or-added lines inside the current diff, and it fits one hunk. Everything else — design findings, `q` questions, multi-file or multi-hunk fixes, deleted-line targets, out-of-diff spans — stays a prose comment stating the fix. See [ADR: suggestion-first-findings](../adrs/suggestion-first-findings.md).

### <a id="fr2"></a>FR2 — Fix payload and assembly
A suggestion is assembled by the agent that has the target file open: replacement lines verbatim, exact leading whitespace, span = anchored line plus range. The existing answering-flow rule ("read the actual file before proposing a suggestion, never from comment text alone") extends to the reviewer side. The [`ni:reviewer`](../../../agents/reviewer.md) output contract stays one-line prose; the posting step re-reads the file and builds the block. See [ADR: suggestion-assembly](../adrs/suggestion-assembly.md).

### <a id="fr3"></a>FR3 — Platform posting payloads
Both reference files gain a "Posting a suggestion" section with the exact working payloads:
- [`code-review/gitlab.md`](../../../skills/code-review/gitlab.md): diff discussion via `glab api` (`position[base_sha|head_sha|start_sha]` from fresh `diff_refs`, `new_path`, `new_line`), fence `suggestion:-N+M` (cap 201 lines), anchor on `new_line` only, note the 60 notes/minute cap, stale `diff_refs` cause 400.
- [`code-review/github.md`](../../../skills/code-review/github.md): one review call (`POST /repos/{owner}/{repo}/pulls/{number}/reviews` with `comments[]`) carrying all inline comments, `start_line`/`start_side` + `line`/`side=RIGHT`, bare `suggestion` fence, `commit_id` = current head SHA; deleted lines and file-level comments cannot carry suggestions; the reading query gains `startLine` and `diffSide` so replies know the anchored span.
Reviewee-side apply facts land where relevant: GitLab `PUT /suggestions/:id/apply` and `batch_apply` (single apply credits the suggester as author); GitHub UI-only.

### <a id="fr4"></a>FR4 — Review-loop posts applicable fixes
[`review-loop`](../../../commands/review-loop.md) step 5 changes from "post each finding as its own discussion with `file:line` in the body" to: anchor every finding on the diff; attach the fix as a suggestion when FR1 says so; prose fallback otherwise; batch per platform rule (single review call on GitHub, serialised notes under the rate cap on GitLab) — stated platform-neutrally with the mechanics in the reference files.

### <a id="fr5"></a>FR5 — De-platform the agnostic templates
The `suggestion:-0+0` GitLab syntax leaves [`code-review/SKILL.md`](../../../skills/code-review/SKILL.md) (lines 196-209, 253-256): the Suggestion column and markdown template become platform-neutral ("suggestion with range per the platform file"), and the range syntax pointer already at line 217 becomes the single reference.

### <a id="fr6"></a>FR6 — Thread auto-resolution on re-review
On a later loop iteration over a change with new commits, the reviewer checks each of its own unresolved threads against the current code: when the posted fix, or an equivalent removing the defect, is present, it replies naming the fixing commit and resolves the thread. Own threads only; ambiguous stays open; no nag replies. Resolve commands already exist in both reference files. See [ADR: thread-auto-resolution](../adrs/thread-auto-resolution.md) (accepted — user decision, 2026-09-28).

## Non-Functional Requirements

### <a id="nfr1"></a>NFR1 — Plugin validates
- **Scenario**: after every task → plugin valid
- **Measure**: `claude plugin validate .` exits 0
- **Verify**: `claude plugin validate .`

### <a id="nfr2"></a>NFR2 — Size limits hold
- **Scenario**: edited skill files → ni limits hold
- **Measure**: every touched [skill file](../../../skills/) under 300 lines
- **Verify**: `awk 'END{exit NR>=300}' skills/code-review/SKILL.md && awk 'END{exit NR>=300}' skills/code-review/gitlab.md && awk 'END{exit NR>=300}' skills/code-review/github.md`

### <a id="nfr3"></a>NFR3 — No platform syntax leak
- **Scenario**: after FR5 → GitLab range syntax lives only in its platform file
- **Measure**: zero `suggestion:-` hits outside [`code-review/gitlab.md`](../../../skills/code-review/gitlab.md)
- **Verify**: `! grep -rn "suggestion:-" skills/ commands/ --include='*.md' | grep -v "code-review/gitlab.md"`

## Non-goals

- **Auto-applying suggestions.** The point is reviewee consensus: the human clicks apply. GitLab's apply API is documented in the reference file for completeness, but no loop calls it. Revisit if a trusted-fix policy emerges.
- **GitHub suggestions on unchanged lines.** September 2025 preview, REST creation still limited — a bot cannot rely on it. Documented as a gap in github.md.
- **Pushing fix commits to the reviewee's branch** (GitHub's only programmatic apply path). Outside the reviewer role.
- **`glab mr diff-comment` native command** — still an open upstream issue (gitlab-org/cli#8172); `glab api` is the route.

## Rabbit holes

- **Whitespace fidelity.** Neither platform normalises indentation; a mismatch makes the suggestion noisy or dead. Cap: replacement lines are copied from the real file and edited, never regenerated from memory.
- **Multi-hunk or repeated fixes.** Cap: one suggestion per contiguous span; a fix spanning hunks falls back to prose. No stitching.
- **Anchor freshness.** Cap: fetch `diff_refs` / head SHA immediately before posting; on a 400/outdated error, refetch once, then fall back to prose.

## Failure modes

| Failure | Detection | Response | Blast radius |
|---|---|---|---|
| Suggestion rejected (stale anchor) | API 400 / comment marked outdated | refetch refs, retry once, then post prose fallback | one finding |
| Fix targets a deleted line | FR1 classification | prose comment; never a suggestion fence | one finding |
| Rate limit hit (GitLab 60 notes/min, GitHub secondary) | 429 / secondary-limit error | serialise, wait, resume; GitHub already batched in one call | one loop run |
| Suggestion fence in a non-diff note | FR1 anchoring rule violated | forbidden by rule; verify greps template leaks | rendering only |

## Design

Text-only change to three existing files plus [`review-loop`](../../../commands/review-loop.md). Ownership unchanged from [platform-reference-files](../../20260928_gh-cli-integration/adrs/platform-reference-files.md): the flow and classification in [`code-review/SKILL.md`](../../../skills/code-review/SKILL.md), platform payloads in the two reference files, the loop stays agnostic.

Decisions:
- [Suggestion-first findings](../adrs/suggestion-first-findings.md) — when a finding becomes a suggestion, and the fallback ladder
- [Suggestion assembly](../adrs/suggestion-assembly.md) — who builds the block and from what
- [Thread auto-resolution](../adrs/thread-auto-resolution.md) — own threads close when a new commit fixes the finding (evidence-gated)

## Data & migration

N/A: markdown-only.

## Cross-cutting Concerns

Rollout: version bump to 1.5.0 at the release task, README rows for the loops mention applicable suggestions. Rollback: git revert.
