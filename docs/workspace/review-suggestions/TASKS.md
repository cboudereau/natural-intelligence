# review-suggestions — Tasks

Design: [DESIGN.md](./DESIGN.md) · Gate: [PREFLIGHT.md](./PREFLIGHT.md)

## Analysis

Build: N/A — markdown-only plugin
Test: `claude plugin validate .` — verified green (one warning: this workspace's own root [`CLAUDE.md`](../../../CLAUDE.md) anchor, transient until Phase 6)
Lint: link lint (in [PREFLIGHT.md](./PREFLIGHT.md) and the checkpoint)

Environment: neither `gh` nor `glab` installed here; payloads are written from the researched documented behaviour ([suggestion-first-findings](./adrs/suggestion-first-findings.md) sources) and flagged `manual` where only a live call proves them.

### Known-failing tests
| Test | Reason | Action |
|---|---|---|
| `claude plugin validate . --strict` | Transient root [`CLAUDE.md`](../../../CLAUDE.md) | strict at Phase 6 teardown |

### Requirement traceability

| Artifact | Stability | Addresses | Notes |
|---|---|---|---|
| [`code-review/SKILL.md`](../../../skills/code-review/SKILL.md) classification + read-first rule | published | [FR1](./DESIGN.md#fr1), [FR2](./DESIGN.md#fr2), [FR5](./DESIGN.md#fr5) | Rule owner; other files point here |
| [`code-review/gitlab.md`](../../../skills/code-review/gitlab.md) posting-a-suggestion section | published | [FR3](./DESIGN.md#fr3) | position payload, `suggestion:-N+M`, caps |
| [`code-review/github.md`](../../../skills/code-review/github.md) posting-a-suggestion section | published | [FR3](./DESIGN.md#fr3) | review-batch call, `start_line`/`line`, bare fence |
| [`commands/review-loop.md`](../../../commands/review-loop.md) step 5 | published | [FR4](./DESIGN.md#fr4) | stays platform-agnostic |

### Instruction invariants

| Rule | Where | Invariant |
|---|---|---|
| Classification ladder | [`code-review/SKILL.md`](../../../skills/code-review/SKILL.md) | Suggestion only when mechanical + kept/added lines in diff + one hunk (+ 201-line GitLab cap); else prose with the concrete fix ([ADR](./adrs/suggestion-first-findings.md)) |
| Read-first, both directions | [`code-review/SKILL.md`](../../../skills/code-review/SKILL.md) | No suggestion without the real file open; replacement lines are edited copies ([ADR](./adrs/suggestion-assembly.md)) |
| Anchor freshness, error path | reference files | Stale refs → refetch once, retry once, then prose fallback |
| Deleted lines, error path | reference files | `--old-line` / `side=LEFT` never carries a fence |
| Batching | reference files | GitHub: all inline comments in one review call; GitLab: serialised notes under 60/min |
| Platform syntax containment | agnostic files | `suggestion:-` appears only in [`code-review/gitlab.md`](../../../skills/code-review/gitlab.md) ([NFR3](./DESIGN.md#nfr3)) |
| Loop agnosticism | [`commands/review-loop.md`](../../../commands/review-loop.md) | Still zero `glab`/`gh` invocations (gate from [20260928_gh-cli-integration](../../20260928_gh-cli-integration/README.md) holds) |
| Auto-resolution, error path | [`skills/code-review/SKILL.md`](../../../skills/code-review/SKILL.md) | Own threads only; evidence gate before resolve; ambiguous stays open ([ADR](./adrs/thread-auto-resolution.md)) |

## Tasks

### 1. Classification and read-first rules in code-review ([FR1](./DESIGN.md#fr1), [FR2](./DESIGN.md#fr2), [FR5](./DESIGN.md#fr5), [NFR3](./DESIGN.md#nfr3))
**Goal**: one owner for "when is a finding a suggestion and who builds it"; agnostic file free of GitLab syntax.
**Artifacts**: [`skills/code-review/SKILL.md`](../../../skills/code-review/SKILL.md)
**Constraints**:
- [ADR: suggestion-first-findings](./adrs/suggestion-first-findings.md) — ladder verbatim in intent, severity orthogonal, prose fallback always states the fix
- [ADR: suggestion-assembly](./adrs/suggestion-assembly.md) — generalise the answering-flow read-first rule (line ~181) to both directions; one owner
- Giving-a-review flow gains the posting step it lacks (anchor on diff, suggestion per ladder, prose fallback) — flow only, commands stay in the reference files
- [FR5](./DESIGN.md#fr5): `-0+0`/`-1+2` examples and the ` ```suggestion:-0+0 ` template (lines ~196-209, 253-256) become platform-neutral wording with the existing pointer (line ~217) as the single syntax reference
- Under 300 lines; description updated so "suggestion blocks" is no longer GitLab-only wording
**Tests** (red before the edit): `grep -q "suggestion:-0+0" skills/code-review/SKILL.md` currently succeeds — must fail after.
**Verify**: `! grep -rn "suggestion:-" skills/ commands/ --include='*.md' | grep -v "code-review/gitlab.md" && grep -qi "ladder\|applicable suggestion" skills/code-review/SKILL.md && awk 'END{exit NR>=300}' skills/code-review/SKILL.md && claude plugin validate .`
**Acceptance criteria**:
- [x] Ladder present with all three conditions and the fallback; read-first rule generalised, stated once
- [x] No `suggestion:-` outside gitlab.md; verify exits 0
**Depends on**: (none)
**Time-box**: ~50 min

### 2. GitLab posting payload ([FR3](./DESIGN.md#fr3))
**Goal**: a reviewer can post an applicable suggestion on a GitLab MR with copy-paste commands.
**Artifacts**: [`skills/code-review/gitlab.md`](../../../skills/code-review/gitlab.md)
**Constraints**:
- "Posting a suggestion" section: `glab api` POST to `projects/:id/merge_requests/:iid/discussions` with `position[base_sha|head_sha|start_sha]` from fresh `diff_refs`, `position[position_type]=text`, `position[new_path]`, `position[new_line]`; body fence `suggestion:-N+M`; range cap 201 lines; anchor on `new_line` only, never `--old-line`
- Error paths per the invariants table (stale refs 400 → refetch once; rate cap 60 notes/min → serialise)
- Reviewee-side note: apply endpoints `PUT /suggestions/:id/apply` and `PUT /suggestions/batch_apply` with `commit_message`; single apply credits the suggester as commit author
- Under 300 lines
**Tests** (red before the edit): `grep -q "batch_apply" skills/code-review/gitlab.md` currently fails — must pass after.
**Verify**: `grep -q "position\[base_sha\]" skills/code-review/gitlab.md && grep -q "batch_apply" skills/code-review/gitlab.md && grep -q "201" skills/code-review/gitlab.md && awk 'END{exit NR>=300}' skills/code-review/gitlab.md && claude plugin validate .`
**Acceptance criteria**:
- [x] Payload, caps, error paths, apply facts present; verify exits 0
**Depends on**: task 1
**Time-box**: ~40 min

### 3. GitHub posting payload ([FR3](./DESIGN.md#fr3))
**Goal**: same for GitHub PRs.
**Artifacts**: [`skills/code-review/github.md`](../../../skills/code-review/github.md)
**Constraints**:
- "Posting a suggestion" section: one `gh api` review call (`POST /repos/{owner}/{repo}/pulls/{number}/reviews` with `comments[]`: `path`, `line`, `side=RIGHT`, `start_line`/`start_side` for ranges, body with bare ` ```suggestion ` fence), `commit_id` = current head SHA; batching rationale (one notification, secondary rate limits)
- Constraints stated: deleted lines (`side=LEFT`) and file-level comments cannot carry an applicable fence; unchanged-line commenting is a preview with limited API — do not rely on it; apply is UI-only (no REST endpoint, no GraphQL mutation, 2026); applier is committer, suggester co-author
- Reading query gains `startLine` and `diffSide` so replies know the anchored span
- `manual: no gh binary in this environment` note stays accurate; under 300 lines
**Tests** (red before the edit): `grep -q "startLine" skills/code-review/github.md` currently fails — must pass after.
**Verify**: `grep -q "start_line" skills/code-review/github.md && grep -q "startLine" skills/code-review/github.md && grep -qi "deleted lines" skills/code-review/github.md && awk 'END{exit NR>=300}' skills/code-review/github.md && claude plugin validate .`
**Acceptance criteria**:
- [x] Payload, batch call, constraints, apply facts present; verify exits 0
**Depends on**: task 1
**Time-box**: ~40 min

### 4. Review-loop posts applicable fixes ([FR4](./DESIGN.md#fr4))
**Goal**: loop findings arrive one-click applicable where the ladder allows.
**Artifacts**: [`commands/review-loop.md`](../../../commands/review-loop.md)
**Constraints**:
- Step 5 rewritten: anchor every finding on the diff; attach the fix as a suggestion when the classification ladder says so; prose fallback otherwise; batch per the platform reference file
- Platform-agnostic: still zero `glab`/`gh` invocations and zero platform fields; ladder and payloads referenced, not restated
- Keep-it-simple note (user decision, 2026-09-28): the loop orchestrates only — the review itself follows the [`code-review`](../../../skills/code-review/SKILL.md) principles (finding format, disposition rules, red flags, no scope creep); one concern per thread; a suggestion is a posting mechanism, never a licence for bigger rewrites
**Tests** (red before the edit): `grep -qi "suggestion" commands/review-loop.md` currently fails — must pass after.
**Verify**: `grep -qi "suggestion" commands/review-loop.md && ! grep -E "(glab|gh) " commands/review-loop.md && ! grep -q "suggestion:-" commands/review-loop.md && claude plugin validate .`
**Acceptance criteria**:
- [x] Step 5 carries the anchor + suggestion + fallback + batch instruction; loop still agnostic; verify exits 0
**Depends on**: task 1, task 2, task 3
**Time-box**: ~30 min

### 5. Thread auto-resolution on re-review ([FR6](./DESIGN.md#fr6))
**Goal**: fixed findings close themselves; the reviewee never chases the bot.
**Artifacts**: [`skills/code-review/SKILL.md`](../../../skills/code-review/SKILL.md), [`commands/review-loop.md`](../../../commands/review-loop.md)
**Constraints**:
- [ADR: thread-auto-resolution](./adrs/thread-auto-resolution.md) — own threads only; evidence gate (re-read span at new head + diff since posting) before any resolve; reply naming the fixing commit, then resolve; ambiguous stays open; no nag replies
- The rule lives in [`skills/code-review/SKILL.md`](../../../skills/code-review/SKILL.md) (one owner, next to the read-first rule); the loop gains a re-review step referencing it
- Loop stays agnostic: resolve commands are already in the reference files; the loop names neither
**Tests** (red before the edit): `grep -qi "auto-resol\|resolve its own" commands/review-loop.md` currently fails — must pass after.
**Verify**: `grep -qi "resolv" commands/review-loop.md && grep -qi "own thread" skills/code-review/SKILL.md && ! grep -E "(glab|gh) " commands/review-loop.md && awk 'END{exit NR>=300}' skills/code-review/SKILL.md && claude plugin validate .`
**Acceptance criteria**:
- [x] Rule present with scope, evidence gate, reply-then-resolve order
- [x] Loop re-review step present; loop still agnostic; verify exits 0
**Depends on**: task 1
**Time-box**: ~40 min

### 6. Version bump and README ([NFR1](./DESIGN.md#nfr1))
**Goal**: release readiness, no tagging (standing user decision).
**Artifacts**: [`README.md`](../../../README.md), [`.claude-plugin/plugin.json`](../../../.claude-plugin/plugin.json)
**Constraints**:
- Version `1.4.0` to `1.5.0`; nothing else in plugin.json
- README: `ni:code-review` and `/ni:review-loop` rows mention one-click applicable suggestions on both platforms and auto-resolution of fixed threads
**Tests** (red before the edit): `grep -q '"version": "1.5.0"' .claude-plugin/plugin.json` currently fails — must pass after.
**Verify**: `grep -q '"version": "1.5.0"' .claude-plugin/plugin.json && grep -qi "suggestion" README.md && claude plugin validate .`
**Acceptance criteria**:
- [ ] Version 1.5.0; README rows updated; verify exits 0
**Depends on**: tasks 1–5
**Time-box**: ~20 min

## Sessions

### Session 1 — Applicable suggestions (~3.5H)
Tasks: 1, 2, 3, 4, 5, 6
**Skills**: `skill` (ni conventions), `git-conventions` (commits), `evidence-based-analysis` (citations)
**Checkpoint**: `claude plugin validate . && ! grep -rn "suggestion:-" skills/ commands/ --include='*.md' | grep -v "code-review/gitlab.md" && grep -qi "suggestion" commands/review-loop.md && ! grep -E "(glab|gh) " commands/review-loop.md commands/merge-loop.md && ! grep -rPn '(?<!\[)\x60(?:[\w.-]+/)*[\w.-]+\.md(?::\d+(?:[-,:]\d+)?)?\x60' docs/workspace/review-suggestions --include='*.md'`
**Commit point**: yes — one commit per task

## Quality gates (post-session review)
- [ ] Acceptance criteria green
- [ ] Code review: edits match [DESIGN.md](./DESIGN.md) intent; ladder and read-first rule have one owner
- [ ] Organization: no reference-to-reference links; links relative and resolving
- [ ] Cross-referencing: link lint green
- [ ] Security: no tokens in payload examples; auth rules untouched
- [ ] Command correctness: payloads match the researched docs; `manual` flags where unverifiable
