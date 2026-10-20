<p align="center"><img src="assets/logo.svg" alt="ni" width="300"></p>

[![Linux](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/linux.yml/badge.svg)](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/linux.yml) [![Windows](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/windows.yml/badge.svg)](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/windows.yml) [![macOS](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/macos.yml/badge.svg)](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/macos.yml)

# ni

**Never Claude alone.** You and I: Claude NI.

**ni** stands for natural intelligence, as opposed to artificial. Say it *nickel* (French: spot on, good enough) or *nice* (UK/US). Either way, it is the human staying in the loop.

ni is a Claude Code plugin, distributed through the [itsaspacestation marketplace](https://github.com/itsaspacestation/claude-marketplace). Its `skills/` folder also works as a plain skills source for Copilot CLI, Cursor, and Gemini CLI.

## Benchmark vs other plugins (run by ni's author)

Medians of 3 trials per scenario, [ni-bench](https://github.com/itsaspacestation/ni-bench), claude-sonnet-5-5 as subject and as judge, Claude Code 2.1.285, 2026-10-20. ni's author wrote and runs ni-bench, its scenarios, its criteria, and the judge rubric. ni 2.2.0+local.e472763 is measured from a local checkout, not a tagged release; the other plugins are their marketplace releases. ni runs `ni:full`; the other arms run their default output style, so the token, cost, and verbosity rows compare a tuned configuration with defaults. Resource percentages are relative to the best arm (100%). 🏆 marks the best plugin only where the gap exceeds the judge's own drift (5 points; 10 on a build scenario, where the subject's own run-to-run variance adds as much) or 10 points on a resource row; the no-plugin baseline is cheaper and faster than every plugin on both scenarios below. The two scenarios below are the ones ported from superpowers-evals; the five home-grown scenarios, written by ni's author, are under Latest benchmark. Judge rubric v2.2 reads the commit log and scores whether tasks and commits are end-to-end slices and whether a build run finishes without deferring a step to the user; ni adopted a slice rule and an unconditional per-task commit the same days, so those items favour it until another plugin follows. The debug agent_executability rows score the red-to-green trace an arm pastes into its final message, not the tool log, and favour ni for the same reason (see the ni-bench CALIBRATION notes). The build gain reproduces in three consecutive runs (run-20261019-193123, run-20261020-092358, this one).

### ported-build (superpowers-evals)

| KPI | baseline | openspec 1.14.2 | superpowers 7.0.0 | ni 2.2.0+local.e472763 | mattpocock 1.3.1 |
|---|---|---|---|---|---|
| tokens_total | 100% (1 952 tok) | 42% (4 629 tok) | 52% (3 786 tok) | 40% (4 879 tok) | 39% (5 048 tok) |
| cost_usd | 100% ($0.0602) | 36% ($0.1655) | 🏆52% ($0.1161) | 41% ($0.1474) | 23% ($0.2601) |
| duration_s | 100% (15.4 s) | 37% (41.1 s) | 🏆57% (27.1 s) | 38% (40.7 s) | 30% (51.4 s) |
| turns | 100% (5) | 50% (10) | 50% (10) | 42% (12) | 29% (17) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 62% (62) | 70% (70) | 72% (72) | 80% (80) | 70% (70) |
| agent_executability | 35% (35) | 40% (40) | 55% (55) | 🏆82% (82) | 55% (55) |
| verbosity_score | 85% (85) | 72% (72) | 82% (82) | 85% (85) | 70% (70) |
| outcome | 100% (3/3) | 67% (2/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

### ported-debug (superpowers-evals)

| KPI | baseline | openspec 1.14.2 | superpowers 7.0.0 | ni 2.2.0+local.e472763 | mattpocock 1.3.1 |
|---|---|---|---|---|---|
| tokens_total | 100% (581 tok) | 🏆88% (663 tok) | 53% (1 101 tok) | 55% (1 064 tok) | 53% (1 087 tok) |
| cost_usd | 100% ($0.0342) | 🏆95% ($0.0362) | 44% ($0.0774) | 51% ($0.0667) | 50% ($0.0690) |
| duration_s | 100% (7.8 s) | 🏆87% (9.0 s) | 59% (13.2 s) | 58% (13.5 s) | 55% (14.2 s) |
| turns | 100% (3) | 🏆100% (3) | 50% (6) | 50% (6) | 75% (4) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 80% (80) | 82% (82) | 88% (88) | 90% (90) | 88% (88) |
| agent_executability | 45% (45) | 55% (55) | 60% (60) | 🏆82% (82) | 62% (62) |
| verbosity_score | 88% (88) | 88% (88) | 88% (88) | 88% (88) | 85% (85) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |
### Reply styles vs built-in styles

| style | output tokens | reply chars | facts kept | words |
|---|---|---|---|---|
| default | 4911 | 9175 | 36/36 | 1494 |
| concise | 5286 | 10146 | 36/36 | 1617 |
| ni:lite | 3374 | 5618 | 36/36 | 899 |
| ni:full | 3271 | 3445 | 36/36 | 521 |

Setup: 10 prompts, sonnet replies, haiku judge, Claude Code 2.1.292, 2026-10-07, `scripts/style-bench.sh` in [ni-bench](https://github.com/itsaspacestation/ni-bench). ni:lite is 36% under concise on output tokens, ni:full 39% under ni:lite on visible characters, all facts kept.

## Quick tour
Claude does the heavy lifting. You make the calls. Skills trigger on their own from what you ask; the prompts below are examples.

### 1. Set up
Install as above, run `/reload-plugins`, then `/ni:help` to check the skills are loaded.

### 2. Claude plans, and I decide
> /ni:plan a workspace for the invoice export feature.

or simply
> Plan a workspace for the invoice export feature.

`ni:plan` explores the codebase and drafts `DESIGN.md`, ADRs, and `TASKS.md` under `docs/workspace/<name>/`. You spend about 30 to 40 minutes reviewing scope, settling the ADRs, and approving the tasks. Then say go: Claude runs the tasks on autopilot and stops only when a decision falls outside the plan.

For a small change, Claude Code's built-in plan mode is enough.

### 3. Claude builds, and I steer
> Fix the rounding bug in the VAT total.

`ni:software-engineer`, `ni:tdd`, and `ni:debug` enforce plan, failing test, fix, and commit, with the root cause found before any fix. `ni:git-conventions` writes the commit and the MR or PR description, routing to the right forge from the origin remote. Nothing is pushed without your go.

### 4. Claude reviews, and I judge
> /ni:code-review my branch before I open the MR.

or simply

> Review my branch before I open the MR.

`ni:code-review` checks design, tests, performance, security, and correctness, and cites every finding by file and line. You decide what to fix.

### 5. Claude reviews others, and I sign off
```
/ni:review-loop group/project
```
Works on GitLab and GitHub: pass a project path or URL, or let it read the origin remote. Reviews every MR or PR assigned to you in a loop, posts each finding as its own discussion, and reports the links. It never approves, merges, or resolves: those stay yours.

### 6. Claude merges mine once approved
```
/ni:merge-loop group/project
```
Watches your own MRs or PRs in a loop and merges each one once every reviewer has approved, threads are resolved, and CI is green. Anything blocked is reported with its reason, never forced.

### 7. Claude answers reviewers, and I approve
> Address the unresolved threads on MR !42 (or PR #42).

`ni:code-review` reads the threads, drafts the fixes and replies, and shows you a preview. Nothing is posted or resolved until you approve it.

## Requirements
- Claude Code with its own [system requirements](https://code.claude.com/docs/en/setup) per OS: macOS, Linux, or Windows (native or WSL).
- Claude Code 2.1.251 or later for live output-style switching with `/output-style`.
- `glab` for GitLab or `gh` for GitHub, depending on the forge the project uses.
- VS Code with the draw.io extension, or draw.io, only to render and export c4-graph output.

ni ships no scripts and assumes no shell: the commands in the skills are shell-neutral and run in bash, PowerShell, and cmd.

## Install
Inside Claude Code:
```
/plugin marketplace add itsaspacestation/claude-marketplace
/plugin install ni@itsaspacestation
```

Or from a shell:
```bash
claude plugin marketplace add itsaspacestation/claude-marketplace
claude plugin install ni@itsaspacestation            # user scope, every project
claude plugin install ni@itsaspacestation -s project # this project only, written to .claude/settings.json
```

Run `/reload-plugins` or start a new session. `/ni:help` lists the skills.

For a team, commit this to the project's `.claude/settings.json`. Claude Code offers the install on first launch:
```json
{
  "extraKnownMarketplaces": {
    "itsaspacestation": { "source": { "source": "github", "repo": "itsaspacestation/claude-marketplace" } }
  },
  "enabledPlugins": { "ni@itsaspacestation": true }
}
```

For other agents, copy `skills/` into `~/.copilot/`, `~/.cursor/`, or `~/.gemini/` (Windows: `%USERPROFILE%\.copilot`, `%USERPROFILE%\.cursor`, `%USERPROFILE%\.gemini`).

## Reply styles
ni ships two output styles. Switch live inside Claude Code:
```
/output-style ni:lite
/output-style ni:full
/output-style default
```

- `ni:lite`: no filler, full sentences, every technical fact kept, a scope budget per reply.
- `ni:full`: the same budget, and it also drops articles and allows fragments.
- `default`: Claude Code's built-in style.

The choice persists as `outputStyle`: per project in `.claude/settings.local.json`, or for every project in `~/.claude/settings.json` (Windows: `%USERPROFILE%\.claude\settings.json`). Live switching needs Claude Code 2.1.251 or later.

## Layout
| Path | Purpose |
|---|---|
| `.claude-plugin/plugin.json` | Plugin manifest, name `ni` |
| `agents/` | Subagents with compressed output, spawned as `ni:<agent>` |
| `commands/` | Slash commands, invoked as `/ni:<command>` |
| `skills/` | The skills, invoked as `ni:<skill>` |

## Update
Auto-update is off by default for third-party marketplaces. Turn it on in `/plugin`, under **Marketplaces**, or update by hand:
```bash
claude plugin marketplace update itsaspacestation
claude plugin update ni@itsaspacestation
```
Restart Claude Code to apply.

## Develop
```bash
claude plugin validate . --strict
claude --plugin-dir .   # load from the working tree
bash .github/scripts/docs.test.sh   # docs lint, runs in CI
```

## Release
Users only get an update when `version` in `.claude-plugin/plugin.json` changes.

1. Bump `version` (semver) and commit.
2. `claude plugin validate . --strict`
3. `claude plugin tag .` creates the `ni--v<version>` tag, then push the commit and the tag.

The marketplace entry tracks the default branch, so the marketplace repository needs no change for a release.

### 2.2.0

- `plan`: an ADR is written only when the decision is hard to reverse, surprising later, and weighed against alternatives.
- `plan`: C4, STRIDE, and sequence diagrams only when the system calls for them.
- `plan`: Phase 3 opens with a decision round when a human is present; delegated runs still decide alone.
- `plan`: a root `GLOSSARY.md`, written at integration, read at onboarding.
- `plan`: every task is a vertical slice, one behaviour across every layer it touches, verifiable alone; a wide mechanical change goes expand, migrate, contract.
- `git-conventions`: a change description carries a visual summary, before and after evidence, a reversibility call, and the checklist.
- `debug`: a tight reproduction loop before any hypothesis, a minimise phase, tagged probes.
- `tdd`: the acceptance test enters through the public surface.
- `plan`: one task at a time, each closed by its own commit, headless included; no batch of tests first, no boxes ticked at the end; a test green on its first run folds into the previous commit, never a tests-only commit.
- `plan`: the small route's plan document stays untracked; task commits carry code and tests only.
- `plan`, `tdd`: an implementation covers its own failing test only, so the next slice goes red; the complex route commits the plan at pre-flight in every context, without asking.
- `git-conventions`: stage by path, never `git add -A`; generated files are never committed.
- `plan`: a `Fit` field per task, `closed` or `open`; autopilot starts only when every task is closed, and a closed task runs on a cheaper subagent model.
- `plan`: a task subagent returns `done`, `blocked: <question>`, or `gap: <adr>`; it never pauses or asks the user.
- `code-review`: an Intent category, items 37 to 40, checks the diff against the request.
- `code-review`: item 40 checks that each commit lands one behaviour end to end.
- `retro`: new skill; turns a session's friction into checks, conventions, or pointers.

## Skills
| Skill | Use when |
|---|---|
| `ni:software-engineer` | Implementing, fixing, or refactoring with the plan, test, implement, commit workflow; onboarding on an existing project: commands discovered from the CI pipeline, wrappers, lock files, and repository conventions |
| `ni:tdd` | Writing tests first, red-green-refactor |
| `ni:debug` | Any failure or bug, before proposing a fix |
| `ni:plan` | Multi-session work with a durable workspace, design doc, and ADRs |
| `ni:git-conventions` | Any git operation, commit messages, MR or PR descriptions; routes to the forge from the origin remote and keeps the git-versus-forge boundary |
| `ni:code-review` | Reviewing a change or answering reviewer comments, GitLab and GitHub threads via glab and gh included; posts findings as one-click applicable suggestions on both forges and resolves its own threads once a new commit fixes the finding |
| `ni:retro` | After a session or a plan: turns friction into checks, conventions, or pointers; never touches product code |
| `ni:evidence-based-analysis` | Any claim about the codebase, cited by file and line |
| `ni:bias-analysis` | Comparative studies from field reports, reviews, or statistics: bias checklist sweep with verdicts |
| `ni:skill` | Creating or editing a ni skill, agent, or command |
| `ni:c4-graph` | C4 architecture diagrams as draw.io files on the C4 palette with line jumps on crossings, rendered in VS Code or draw.io and embedded as `.drawio.svg`; converts text, DOT, and draw.io sources |

Run `/ni:help` inside Claude for the same list.

## Commands
User-invoked only; none loads on its own.

| Command | Does |
|---|---|
| `/ni:help` | List the ni skills |
| `/ni:review-loop` | Review MRs or PRs assigned to me in a /loop on GitLab or GitHub, post findings as one-click applicable suggestions where possible, resolve its own threads once a new commit fixes the finding, report the links |
| `/ni:merge-loop` | Merge my MRs or PRs approved by every reviewer in a /loop, report merged and blocked ones |

## Agents
Subagent results land in the main context verbatim, so these three return structured one-liners instead of prose.

| Agent | Use for | Returns |
|---|---|---|
| `ni:investigator` | Where is X defined, what calls Y, map this directory | `path:line - symbol - note` rows |
| `ni:builder` | Surgical edit of 1 or 2 known files | Diff receipt, or `too-big.` / `ambiguous.` |
| `ni:reviewer` | Findings-only review of a diff, branch, or file | `path:L42: severity: problem. fix.` rows |

Rule of thumb: want the result in a third of the tokens, pick ni. Want prose, pick the vanilla agent.

## Latest benchmark
benchmark tool: [ni-bench](https://github.com/itsaspacestation/ni-bench)

### Latest report
run-20261020-110011, 2026-10-20, n=3 (medians; judge drift 2.2 and 3.1 points on identical input, so gaps under 5 are ties), subject and judge claude-sonnet-5-5, simulator claude-haiku-5-5; ni 2.2.0+local.e472763 runs `ni:full`, the other arms run defaults. ni's author runs the benchmark and wrote the five home-grown scenarios and the rubric. Judge rubric v2.2 reads the commit log and scores whether tasks and commits are end-to-end slices and whether a build run finishes without deferring a step to the user; ni adopted a slice rule and an unconditional per-task commit the same days, so those items favour it until another plugin follows. The debug agent_executability rows score the red-to-green trace an arm pastes into its final message, not the tool log, and favour ni for the same reason (see the ni-bench CALIBRATION notes). The build gain reproduces in three consecutive runs (run-20261019-193123, run-20261020-092358, this one).

#### plan-easy (home-grown)

| KPI | baseline | openspec | superpowers | ni 2.2.0+local.e472763 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 100% (2 222 tok) | 59% (3 778 tok) | 53% (4 167 tok) | 98% (2 265 tok) | 57% (3 897 tok) |
| cost_usd | 100% ($0.0643) | 53% ($0.1210) | 53% ($0.1211) | 72% ($0.0897) | 41% ($0.1580) |
| duration_s | 100% (19.1 s) | 56% (33.9 s) | 60% (31.9 s) | 100% (19.1 s) | 61% (31.1 s) |
| turns | 100% (4) | 57% (7) | 67% (6) | 57% (7) | 50% (8) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 0% (1) |
| plan_words | 72% (659 words) | 62% (769 words) | 44% (1 072 words) | 100% (476 words) | 81% (589 words) |
| machine_words | 0 words | 170 words | 0 words | 0 words | 0 words |
| human_readability | 90% (90) | 82% (82) | 88% (88) | 88% (88) | 88% (88) |
| agent_executability | 86% (86) | 76% (76) | 85% (85) | 87% (87) | 80% (80) |
| verbosity_score | 85% (85) | 78% (78) | 78% (78) | 88% (88) | 85% (85) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### plan-complex (home-grown)

| KPI | baseline | openspec | superpowers | ni 2.2.0+local.e472763 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 100% (8 388 tok) | 79% (10 563 tok) | 78% (10 730 tok) | 37% (22 474 tok) | 82% (10 232 tok) |
| cost_usd | 100% ($0.1396) | 52% ($0.2676) | 61% ($0.2279) | 30% ($0.4584) | 52% ($0.2688) |
| duration_s | 100% (66.7 s) | 82% (81.7 s) | 89% (74.7 s) | 40% (164.8 s) | 82% (81.4 s) |
| turns | 100% (3) | 33% (9) | 75% (4) | 23% (13) | 27% (11) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 0% (1) |
| plan_words | 68% (2 664 words) | 84% (2 143 words) | 63% (2 851 words) | 67% (2 701 words) | 100% (1 801 words) |
| machine_words | 0 words | 500 words | 0 words | 3 171 words | 614 words |
| human_readability | 88% (88) | 82% (82) | 88% (88) | 90% (90) | 82% (82) |
| agent_executability | 62% (62) | 62% (62) | 70% (70) | 90% (90) | 60% (60) |
| verbosity_score | 80% (80) | 80% (80) | 78% (78) | 78% (78) | 80% (80) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### debug-easy (home-grown)

| KPI | baseline | openspec | superpowers | ni 2.2.0+local.e472763 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 100% (286 tok) | 75% (381 tok) | 40% (724 tok) | 39% (742 tok) | 42% (682 tok) |
| cost_usd | 100% ($0.0306) | 94% ($0.0324) | 43% ($0.0716) | 51% ($0.0605) | 55% ($0.0559) |
| duration_s | 100% (6.0 s) | 82% (7.3 s) | 65% (9.3 s) | 66% (9.2 s) | 66% (9.1 s) |
| turns | 100% (3) | 75% (4) | 43% (7) | 50% (6) | 75% (4) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 70% (70) | 70% (70) | 85% (85) | 92% (92) | 85% (85) |
| agent_executability | 35% (35) | 35% (35) | 45% (45) | 80% (80) | 55% (55) |
| verbosity_score | 90% (90) | 90% (90) | 90% (90) | 85% (85) | 85% (85) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### debug-complex (home-grown)

| KPI | baseline | openspec | superpowers | ni 2.2.0+local.e472763 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 100% (837 tok) | 97% (866 tok) | 68% (1 236 tok) | 64% (1 312 tok) | 62% (1 356 tok) |
| cost_usd | 100% ($0.0463) | 100% ($0.0463) | 55% ($0.0847) | 59% ($0.0778) | 57% ($0.0817) |
| duration_s | 95% (10.0 s) | 100% (9.5 s) | 65% (14.5 s) | 70% (13.7 s) | 57% (16.7 s) |
| turns | 100% (5) | 100% (5) | 71% (7) | 71% (7) | 83% (6) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 85% (85) | 85% (85) | 88% (88) | 90% (90) | 88% (88) |
| agent_executability | 40% (40) | 55% (55) | 72% (72) | 85% (85) | 70% (70) |
| verbosity_score | 85% (85) | 88% (88) | 90% (90) | 85% (85) | 85% (85) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### build-small (home-grown)

| KPI | baseline | openspec | superpowers | ni 2.2.0+local.e472763 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 100% (1 347 tok) | 85% (1 590 tok) | 48% (2 781 tok) | 34% (3 908 tok) | 37% (3 668 tok) |
| cost_usd | 100% ($0.0555) | 93% ($0.0598) | 44% ($0.1272) | 40% ($0.1378) | 25% ($0.2241) |
| duration_s | 100% (12.1 s) | 81% (14.9 s) | 51% (23.9 s) | 34% (35.3 s) | 25% (48.5 s) |
| turns | 100% (5) | 100% (5) | 50% (10) | 42% (12) | 31% (16) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 72% (72) | 62% (62) | 80% (80) | 80% (80) | 72% (72) |
| agent_executability | 45% (45) | 40% (40) | 55% (55) | 82% (82) | 62% (62) |
| verbosity_score | 85% (85) | 78% (78) | 80% (80) | 85% (85) | 78% (78) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### ported-debug (superpowers-evals)

| KPI | baseline | openspec | superpowers | ni 2.2.0+local.e472763 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 100% (581 tok) | 88% (663 tok) | 53% (1 101 tok) | 55% (1 064 tok) | 53% (1 087 tok) |
| cost_usd | 100% ($0.0342) | 95% ($0.0362) | 44% ($0.0774) | 51% ($0.0667) | 50% ($0.0690) |
| duration_s | 100% (7.8 s) | 87% (9.0 s) | 59% (13.2 s) | 58% (13.5 s) | 55% (14.2 s) |
| turns | 100% (3) | 100% (3) | 50% (6) | 50% (6) | 75% (4) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 80% (80) | 82% (82) | 88% (88) | 90% (90) | 88% (88) |
| agent_executability | 45% (45) | 55% (55) | 60% (60) | 82% (82) | 62% (62) |
| verbosity_score | 88% (88) | 88% (88) | 88% (88) | 88% (88) | 85% (85) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### ported-build (superpowers-evals)

| KPI | baseline | openspec | superpowers | ni 2.2.0+local.e472763 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 100% (1 952 tok) | 42% (4 629 tok) | 52% (3 786 tok) | 40% (4 879 tok) | 39% (5 048 tok) |
| cost_usd | 100% ($0.0602) | 36% ($0.1655) | 52% ($0.1161) | 41% ($0.1474) | 23% ($0.2601) |
| duration_s | 100% (15.4 s) | 37% (41.1 s) | 57% (27.1 s) | 38% (40.7 s) | 30% (51.4 s) |
| turns | 100% (5) | 50% (10) | 50% (10) | 42% (12) | 29% (17) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 62% (62) | 70% (70) | 72% (72) | 80% (80) | 70% (70) |
| agent_executability | 35% (35) | 40% (40) | 55% (55) | 82% (82) | 55% (55) |
| verbosity_score | 85% (85) | 72% (72) | 82% (82) | 85% (85) | 70% (70) |
| outcome | 100% (3/3) | 67% (2/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |
