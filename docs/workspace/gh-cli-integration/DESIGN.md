# gh-cli-integration — Design Doc

## Context

The ni plugin drives forge work through `glab` only. `skills/code-review/gitlab.md` holds every platform command, and `commands/review-loop.md` plus `commands/merge-loop.md` are hard-wired to GitLab (`argument-hint: "[gitlab project path or URL]"`, `glab` in every step). No file mentions `gh`, and no rule tells an agent which platform a repository is on.

Two problems follow:
1. GitHub repositories get no forge support at all.
2. Agents mix `git` and forge commands because no rule states which operations belong to which tool. Investigation found six mixing points: no MR/PR-create flow (the description ends at `clip.exe`, `git-conventions/SKILL.md:99`), rebase reachable both server-side (`merge-loop.md:27-28`) and locally (colliding with the no-force rule, `git-conventions/SKILL.md:29`), ambiguous diff source for reviews (`review-loop.md:16` vs `gitlab.md:45`), the push rule duplicated in three files with differing wording, `glab`'s implicit current-branch MR lookup undocumented, and a `doc:`/`docs(` commit-type mismatch.

Goal: add a GitHub integration mirroring the glab one, and write down the git-versus-forge boundary so agents stop improvising.

## Functional Requirements

### <a id="fr1"></a>FR1 — Platform routing rule
One rule, owned by [`git-conventions`](../../../skills/git-conventions/SKILL.md), tells the agent which forge CLI a repository uses: read the origin host (`git remote get-url origin`), map `github.com` (and GHES hosts) to `gh` and `gitlab.*` to `glab`, allow an explicit argument or user statement to override, and ask the user when the host matches neither. Skills and commands reference this rule instead of restating it. See [ADR: platform-detection](./adrs/platform-detection.md).

### <a id="fr2"></a>FR2 — GitHub review reference file
`skills/code-review/github.md` mirrors `gitlab.md` section for section: reading PRs and review threads, diff, suggestion syntax, posting replies, resolving threads, verifying, setup and auth. Where GitHub has no CLI verb, the file gives the `gh api` (REST or GraphQL) call, exactly as `gitlab.md` does for thread resolution. Known capability gaps are stated, not hidden. `code-review/SKILL.md` routes to the right reference file via FR1 and its description names GitHub and `gh`.

### <a id="fr3"></a>FR3 — Platform-neutral loops
`commands/review-loop.md` and `commands/merge-loop.md` work on both platforms. Each step states the `glab` and the `gh` form (or points to the reference file), the argument hints say "project path or URL", and the merge gates map GitLab's `detailed_merge_status` to GitHub's `mergeStateStatus`/`reviewDecision`/`statusCheckRollup` equivalents. Server-side branch update replaces `glab mr rebase` on GitHub (`gh api .../update-branch`).

### <a id="fr4"></a>FR4 — Git-versus-forge boundary
[`git-conventions`](../../../skills/git-conventions/SKILL.md) states which tool owns which operation. Forge CLI (`gh`/`glab`) owns everything that lives on the server: MR/PR create and description, MR/PR diff, threads, approvals, CI status, merge, server-side rebase or branch update. `git` owns only local state: stage, commit, branch, local diff, log, worktrees, push. The MR/PR description flow becomes `glab mr create/update --description` or `gh pr create/edit --body`, replacing the `clip.exe` clipboard step. The rebase rule says: rebase through the forge, never locally, which resolves the collision with the no-force rule. See [ADR: forge-first-boundary](./adrs/forge-first-boundary.md).

## Non-Functional Requirements

### <a id="nfr1"></a>NFR1 — Plugin validates
- **Scenario**: repository state after every task → plugin manifest and skills remain valid
- **Measure**: `claude plugin validate .` exits 0 per task. `--strict` fails today only because the workspace resume anchor (root `CLAUDE.md`) triggers a root-context warning; the file is deleted at Phase 6, so the release gate is `claude plugin validate . --strict` after workspace teardown.
- **Verify**: `claude plugin validate .` (per task); `claude plugin validate . --strict` (release gate)

### <a id="nfr2"></a>NFR2 — Size and description limits
- **Scenario**: new and edited skill files → ni conventions hold
- **Measure**: every `SKILL.md` and reference file under 300 lines; every frontmatter description under 1024 characters
- **Verify**: `awk 'END{exit NR>=300}' skills/code-review/github.md && awk 'END{exit NR>=300}' skills/code-review/SKILL.md && awk 'END{exit NR>=300}' skills/git-conventions/SKILL.md`

### <a id="nfr3"></a>NFR3 — No clipboard, no platform leak
- **Scenario**: after FR3/FR4 → no Windows-only or single-platform residue in the touched files
- **Measure**: zero hits for `clip.exe` in skills/ and commands/; zero hits for `gitlab project path` in commands/
- **Verify**: `! grep -rn "clip.exe" skills/ commands/ && ! grep -rin "gitlab project path" commands/`

### <a id="nfr4"></a>NFR4 — Command parity
- **Scenario**: every `glab` capability used by the plugin → a `gh` counterpart or a documented gap in `github.md`
- **Measure**: parity checklist in the FR2 task covers all 10 capability rows (list, view threads, structured notes, diff, suggestion, reply, new inline thread, resolve, verify, auth)
- **Verify**: `manual: parity is a semantic property; the FR2 task carries the checklist as acceptance criteria`

## Non-goals

- **Live end-to-end test against a real GitHub PR.** Requires an authenticated `gh` and a disposable repository; deferred to first real use. Command syntax is verified against `gh` help output during implementation instead.
- **Other forges (Bitbucket, Gitea, Codeberg).** No current need; the FR1 routing rule leaves room (unknown host → ask).
- **Fixing the `doc:` vs `docs(` commit-type mismatch** (`git-conventions/SKILL.md:48` vs `plan/SKILL.md:241,334`). Real defect, separate concern; filed here so it is not lost, fixable in a one-line follow-up.
- **Consolidating the push rule's three duplicated statements** into one owner is included in FR4 (the boundary rewrite touches those lines anyway) — not a non-goal, listed here only to note the duplication is known.

## Rabbit holes

- **GitHub review-thread API.** `gh` has no thread verbs; threads live in GraphQL (`reviewThreads`, `resolveReviewThread`). Cap: use the known GraphQL queries and REST reply endpoint, verify shape against `gh api` docs, do not build pagination beyond `first: 100`.
- **GHES host detection.** Arbitrary GitHub Enterprise hostnames cannot be enumerated. Cap: `github.com` plus `gh auth status` hosts; otherwise the unknown-host rule (ask) applies.
- **Merge-gate semantics mapping.** GitLab's `detailed_merge_status` has no exact GitHub twin. Cap: map to the three GitHub fields (`mergeStateStatus`, `reviewDecision`, `statusCheckRollup`) and keep the loop's conservative rule — skip when not clean, never bypass a gate.

## Failure modes

| Failure | Detection | Response | Blast radius |
|---|---|---|---|
| Forge CLI missing or unauthenticated | `gh auth status` / `glab auth status` non-zero | stop, tell the user the exact login command; never create or read tokens | single command run |
| Origin host matches neither forge | FR1 host mapping falls through | ask the user which forge; never guess | routing decision |
| Repository has no origin remote | `git remote get-url origin` fails | ask the user for the project path or URL | routing decision |
| GraphQL thread mutation rejected (permissions) | `gh api graphql` non-zero | report the thread id and error; leave the thread unresolved; continue with the next | one thread |
| PR not mergeable on GitHub | `mergeStateStatus` not `CLEAN` | skip and report, mirror of the GitLab `detailed_merge_status` gate | one PR |

## Design

Text-only change: three skills/commands edited, one file added. No code, no persistence.

```mermaid
graph LR
    GC[git-conventions/SKILL.md<br/>FR1 routing + FR4 boundary] --> CR[code-review/SKILL.md<br/>routes by FR1]
    CR --> GL[code-review/gitlab.md<br/>glab commands]
    CR --> GH[code-review/github.md NEW<br/>gh commands]
    RL[commands/review-loop.md] --> CR
    ML[commands/merge-loop.md] --> GC
```

Ownership after the change (one rule, one owner):
- Routing rule and git/forge boundary: `git-conventions` (FR1, FR4).
- Review flow: `code-review/SKILL.md`; platform commands: `gitlab.md` / `github.md` (FR2).
- Loop orchestration: the two command files, platform-neutral, deferring commands to the reference files where possible (FR3).

No trust boundary crosses inside the plugin: the forge CLIs hold the credentials. The auth sections keep the existing rule — check status, never create or read tokens.

Decisions:
- [Which GitHub CLI](./adrs/github-cli-choice.md)
- [Platform detection](./adrs/platform-detection.md)
- [Git-versus-forge boundary](./adrs/forge-first-boundary.md)

## Data & migration

N/A: markdown-only change, no persistence.

## Cross-cutting Concerns

Rollout: version bump in `.claude-plugin/plugin.json`, `claude plugin validate . --strict`, README tables updated, trigger test with `--model haiku -p` (one prompt that should route to `github.md`, one that should not trigger `code-review`). Rollback: git revert of the release commit.
