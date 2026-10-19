<p align="center"><img src="assets/logo.svg" alt="ni" width="300"></p>

[![Linux](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/linux.yml/badge.svg)](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/linux.yml) [![Windows](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/windows.yml/badge.svg)](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/windows.yml) [![macOS](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/macos.yml/badge.svg)](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/macos.yml)

# ni

**Never Claude alone.** You and I: Claude NI.

**ni** stands for natural intelligence, as opposed to artificial. Say it *nickel* (French: spot on, good enough) or *nice* (UK/US). Either way, it is the human staying in the loop.

ni is a Claude Code plugin, distributed through the [itsaspacestation marketplace](https://github.com/itsaspacestation/claude-marketplace). Its `skills/` folder also works as a plain skills source for Copilot CLI, Cursor, and Gemini CLI.

## Benchmark vs other plugins (run by ni's author)

Medians of 3 trials per scenario, [ni-bench](https://github.com/itsaspacestation/ni-bench), claude-sonnet-5-5 as subject and as judge, Claude Code 2.1.285, 2026-10-18. ni's author wrote and runs ni-bench, its scenarios, its criteria, and the judge rubric. ni 2.2.0+local.ff4e507 is measured from a local checkout, not a tagged release; the other plugins are their marketplace releases. ni runs `ni:full`; the other arms run their default output style, so the token, cost, and verbosity rows compare a tuned configuration with defaults. Resource percentages are relative to the best arm (100%). 🏆 marks the best plugin only where the gap exceeds the judge's own drift (5 points) or 10 points on a resource row; the no-plugin baseline is cheaper and faster than every plugin on both scenarios below. The two scenarios below are the ones ported from superpowers-evals; the five home-grown scenarios, written by ni's author, are under Latest benchmark. Judge rubric v2.1 reads the commit log and scores whether tasks and commits are end-to-end slices; ni adopted a slice rule the same day, so those items favour it until another plugin follows. The debug agent_executability rows score the red-to-green trace an arm pastes into its final message, not the tool log, and favour ni for the same reason (see the ni-bench CALIBRATION notes).

### ported-build (superpowers-evals)

| KPI | baseline | openspec 1.14.2 | superpowers 7.0.0 | ni 2.2.0+local.ff4e507 | mattpocock 1.3.1 |
|---|---|---|---|---|---|
| tokens_total | 100% (1 979 tok) | 42% (4 714 tok) | 57% (3 479 tok) | 59% (3 349 tok) | 51% (3 851 tok) |
| cost_usd | 100% ($0.0607) | 39% ($0.1568) | 54% ($0.1134) | 55% ($0.1113) | 21% ($0.2857) |
| duration_s | 100% (15.9 s) | 39% (40.2 s) | 64% (24.8 s) | 60% (26.4 s) | 48% (33.1 s) |
| turns | 100% (5) | 45% (11) | 🏆62% (8) | 50% (10) | 33% (15) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 62% (62) | 72% (72) | 68% (68) | 🏆78% (78) | 72% (72) |
| agent_executability | 40% (40) | 45% (45) | 55% (55) | 🏆72% (72) | 55% (55) |
| verbosity_score | 85% (85) | 78% (78) | 78% (78) | 🏆85% (85) | 72% (72) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

### ported-debug (superpowers-evals)

| KPI | baseline | openspec 1.14.2 | superpowers 7.0.0 | ni 2.2.0+local.ff4e507 | mattpocock 1.3.1 |
|---|---|---|---|---|---|
| tokens_total | 100% (571 tok) | 🏆87% (654 tok) | 53% (1 085 tok) | 54% (1 067 tok) | 51% (1 110 tok) |
| cost_usd | 100% ($0.0342) | 🏆97% ($0.0351) | 44% ($0.0780) | 50% ($0.0681) | 49% ($0.0696) |
| duration_s | 100% (7.6 s) | 🏆93% (8.2 s) | 59% (12.9 s) | 58% (13.1 s) | 67% (11.4 s) |
| turns | 100% (3) | 🏆100% (3) | 50% (6) | 50% (6) | 75% (4) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 78% (78) | 80% (80) | 82% (82) | 92% (92) | 88% (88) |
| agent_executability | 45% (45) | 55% (55) | 62% (62) | 🏆85% (85) | 72% (72) |
| verbosity_score | 88% (88) | 88% (88) | 88% (88) | 90% (90) | 88% (88) |
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
- `software-engineer`, `plan`: one commit per task in every context, headless included.
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
run-20261018-181207, 2026-10-18, n=3 (medians; judge drift 2.2 and 3.1 points on identical input, so gaps under 5 are ties), subject and judge claude-sonnet-5-5, simulator claude-haiku-5-5; ni 2.2.0+local.ff4e507 runs `ni:full`, the other arms run defaults. ni's author runs the benchmark and wrote the five home-grown scenarios and the rubric. Judge rubric v2.1 reads the commit log and scores whether tasks and commits are end-to-end slices; ni adopted a slice rule the same day, so those items favour it until another plugin follows. The debug agent_executability rows score the red-to-green trace an arm pastes into its final message, not the tool log, and favour ni for the same reason (see the ni-bench CALIBRATION notes).

#### plan-easy (home-grown)

| KPI | baseline | openspec | superpowers | ni 2.2.0+local.ff4e507 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 92% (2 240 tok) | 43% (4 784 tok) | 41% (5 095 tok) | 100% (2 067 tok) | 41% (5 014 tok) |
| cost_usd | 100% ($0.0642) | 42% ($0.1524) | 23% ($0.2763) | 78% ($0.0821) | 37% ($0.1724) |
| duration_s | 90% (18.8 s) | 41% (41.5 s) | 45% (37.8 s) | 100% (17.0 s) | 45% (38.0 s) |
| turns | 100% (4) | 50% (8) | 50% (8) | 57% (7) | 44% (9) |
| user_turns | 100% (0) | 100% (0) | 0% (1) | 100% (0) | 0% (1) |
| plan_words | 74% (567 words) | 50% (847 words) | 37% (1 131 words) | 100% (421 words) | 80% (527 words) |
| machine_words | 0 words | 214 words | 0 words | 0 words | 149 words |
| human_readability | 90% (90) | 84% (84) | 88% (88) | 88% (88) | 85% (85) |
| agent_executability | 86% (86) | 78% (78) | 85% (85) | 87% (87) | 80% (80) |
| verbosity_score | 86% (86) | 80% (80) | 82% (82) | 90% (90) | 78% (78) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### plan-complex (home-grown)

| KPI | baseline | openspec | superpowers | ni 2.2.0+local.ff4e507 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 100% (7 858 tok) | 67% (11 681 tok) | 67% (11 748 tok) | 38% (20 715 tok) | 62% (12 769 tok) |
| cost_usd | 100% ($0.1325) | 48% ($0.2742) | 63% ($0.2115) | 31% ($0.4236) | 42% ($0.3185) |
| duration_s | 100% (61.9 s) | 65% (95.0 s) | 75% (82.5 s) | 42% (145.8 s) | 62% (99.5 s) |
| turns | 100% (2) | 20% (10) | 50% (4) | 15% (13) | 17% (12) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 0% (1) |
| plan_words | 90% (2 588 words) | 79% (2 964 words) | 74% (3 162 words) | 89% (2 624 words) | 100% (2 332 words) |
| machine_words | 0 words | 722 words | 0 words | 2 746 words | 623 words |
| human_readability | 88% (88) | 88% (88) | 88% (88) | 90% (90) | 88% (88) |
| agent_executability | 62% (62) | 62% (62) | 72% (72) | 88% (88) | 62% (62) |
| verbosity_score | 78% (78) | 80% (80) | 78% (78) | 70% (70) | 82% (82) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### debug-easy (home-grown)

| KPI | baseline | openspec | superpowers | ni 2.2.0+local.ff4e507 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 76% (379 tok) | 100% (289 tok) | 44% (660 tok) | 38% (768 tok) | 36% (801 tok) |
| cost_usd | 96% ($0.0321) | 100% ($0.0308) | 43% ($0.0708) | 52% ($0.0595) | 49% ($0.0622) |
| duration_s | 100% (5.6 s) | 100% (5.6 s) | 58% (9.6 s) | 60% (9.3 s) | 57% (9.7 s) |
| turns | 75% (4) | 100% (3) | 43% (7) | 50% (6) | 60% (5) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 70% (70) | 60% (60) | 88% (88) | 90% (90) | 85% (85) |
| agent_executability | 35% (35) | 25% (25) | 55% (55) | 80% (80) | 55% (55) |
| verbosity_score | 90% (90) | 90% (90) | 92% (92) | 85% (85) | 78% (78) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### debug-complex (home-grown)

| KPI | baseline | openspec | superpowers | ni 2.2.0+local.ff4e507 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 96% (903 tok) | 100% (870 tok) | 70% (1 249 tok) | 66% (1 314 tok) | 74% (1 168 tok) |
| cost_usd | 99% ($0.0471) | 100% ($0.0468) | 55% ($0.0854) | 61% ($0.0770) | 64% ($0.0733) |
| duration_s | 92% (10.6 s) | 100% (9.8 s) | 67% (14.6 s) | 69% (14.2 s) | 74% (13.2 s) |
| turns | 100% (5) | 100% (5) | 71% (7) | 71% (7) | 100% (5) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 80% (80) | 82% (82) | 85% (85) | 88% (88) | 85% (85) |
| agent_executability | 45% (45) | 55% (55) | 60% (60) | 82% (82) | 60% (60) |
| verbosity_score | 85% (85) | 88% (88) | 88% (88) | 85% (85) | 85% (85) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### build-small (home-grown)

| KPI | baseline | openspec | superpowers | ni 2.2.0+local.ff4e507 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 72% (2 100 tok) | 100% (1 521 tok) | 49% (3 100 tok) | 58% (2 605 tok) | 44% (3 451 tok) |
| cost_usd | 81% ($0.0635) | 100% ($0.0515) | 38% ($0.1339) | 51% ($0.1016) | 24% ($0.2146) |
| duration_s | 88% (16.1 s) | 100% (14.2 s) | 56% (25.6 s) | 67% (21.4 s) | 32% (44.4 s) |
| turns | 80% (5) | 100% (4) | 40% (10) | 40% (10) | 29% (14) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 72% (72) | 80% (80) | 78% (78) | 85% (85) | 78% (78) |
| agent_executability | 50% (50) | 60% (60) | 70% (70) | 72% (72) | 62% (62) |
| verbosity_score | 85% (85) | 80% (80) | 80% (80) | 85% (85) | 78% (78) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### ported-debug (superpowers-evals)

| KPI | baseline | openspec | superpowers | ni 2.2.0+local.ff4e507 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 100% (571 tok) | 87% (654 tok) | 53% (1 085 tok) | 54% (1 067 tok) | 51% (1 110 tok) |
| cost_usd | 100% ($0.0342) | 97% ($0.0351) | 44% ($0.0780) | 50% ($0.0681) | 49% ($0.0696) |
| duration_s | 100% (7.6 s) | 93% (8.2 s) | 59% (12.9 s) | 58% (13.1 s) | 67% (11.4 s) |
| turns | 100% (3) | 100% (3) | 50% (6) | 50% (6) | 75% (4) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 78% (78) | 80% (80) | 82% (82) | 92% (92) | 88% (88) |
| agent_executability | 45% (45) | 55% (55) | 62% (62) | 85% (85) | 72% (72) |
| verbosity_score | 88% (88) | 88% (88) | 88% (88) | 90% (90) | 88% (88) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### ported-build (superpowers-evals)

| KPI | baseline | openspec | superpowers | ni 2.2.0+local.ff4e507 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 100% (1 979 tok) | 42% (4 714 tok) | 57% (3 479 tok) | 59% (3 349 tok) | 51% (3 851 tok) |
| cost_usd | 100% ($0.0607) | 39% ($0.1568) | 54% ($0.1134) | 55% ($0.1113) | 21% ($0.2857) |
| duration_s | 100% (15.9 s) | 39% (40.2 s) | 64% (24.8 s) | 60% (26.4 s) | 48% (33.1 s) |
| turns | 100% (5) | 45% (11) | 62% (8) | 50% (10) | 33% (15) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 62% (62) | 72% (72) | 68% (68) | 78% (78) | 72% (72) |
| agent_executability | 40% (40) | 45% (45) | 55% (55) | 72% (72) | 55% (55) |
| verbosity_score | 85% (85) | 78% (78) | 78% (78) | 85% (85) | 72% (72) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |
