<p align="center"><img src="assets/logo.svg" alt="ni" width="300"></p>

[![Linux](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/linux.yml/badge.svg)](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/linux.yml) [![Windows](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/windows.yml/badge.svg)](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/windows.yml) [![macOS](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/macos.yml/badge.svg)](https://github.com/itsaspacestation/natural-intelligence/actions/workflows/macos.yml)

# ni

**Never Claude alone.** You and I: Claude NI.

**ni** stands for natural intelligence, as opposed to artificial. Say it *nickel* (French: spot on, good enough) or *nice* (UK/US). Either way, it is the human staying in the loop.

ni is a Claude Code plugin, distributed through the [itsaspacestation marketplace](https://github.com/itsaspacestation/claude-marketplace). Its `skills/` folder also works as a plain skills source for Copilot CLI, Cursor, and Gemini CLI.

## Benchmark vs other plugin

Medians of 3 trials per scenario, [ni-bench](https://github.com/itsaspacestation/ni-bench), claude-sonnet-5-5, Claude Code 2.1.285, 2026-10-18. ni 2.2.0 runs the `ni:full` output style. Resource percentages are relative to the best arm (100%); 🏆 marks the best plugin.

### ported-build (superpowers-evals)

| KPI | baseline | openspec 1.14.2 | superpowers 7.0.0 | ni 2.2.0 | mattpocock 1.3.1 |
|---|---|---|---|---|---|
| tokens_total | 100% (1 838 tok) | 49% (3 754 tok) | 52% (3 536 tok) | 🏆58% (3 195 tok) | 🏆58% (3 187 tok) |
| cost_usd | 100% ($0.0586) | 37% ($0.1592) | 50% ($0.1162) | 🏆54% ($0.1090) | 22% ($0.2680) |
| duration_s | 100% (14.8 s) | 36% (41.3 s) | 59% (24.9 s) | 🏆61% (24.2 s) | 42% (35.1 s) |
| turns | 100% (5) | 🏆50% (10) | 45% (11) | 🏆50% (10) | 31% (16) |
| user_turns | 100% (0) | 🏆100% (0) | 🏆100% (0) | 🏆100% (0) | 🏆100% (0) |
| human_readability | 55% (55) | 72% (72) | 72% (72) | 🏆80% (80) | 60% (60) |
| agent_executability | 40% (40) | 50% (50) | 70% (70) | 🏆78% (78) | 55% (55) |
| verbosity_score | 85% (85) | 78% (78) | 🏆82% (82) | 🏆82% (82) | 68% (68) |
| outcome | 100% (3/3) | 67% (2/3) | 🏆100% (3/3) | 🏆100% (3/3) | 67% (2/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

### ported-debug (superpowers-evals)

| KPI | baseline | openspec 1.14.2 | superpowers 7.0.0 | ni 2.2.0 | mattpocock 1.3.1 |
|---|---|---|---|---|---|
| tokens_total | 89% (681 tok) | 🏆100% (605 tok) | 57% (1 059 tok) | 51% (1 188 tok) | 51% (1 198 tok) |
| cost_usd | 98% ($0.0353) | 🏆100% ($0.0345) | 45% ($0.0775) | 48% ($0.0720) | 49% ($0.0703) |
| duration_s | 92% (8.2 s) | 🏆100% (7.5 s) | 61% (12.4 s) | 56% (13.4 s) | 63% (12.0 s) |
| turns | 100% (3) | 🏆100% (3) | 50% (6) | 50% (6) | 75% (4) |
| user_turns | 100% (0) | 🏆100% (0) | 🏆100% (0) | 🏆100% (0) | 🏆100% (0) |
| human_readability | 78% (78) | 80% (80) | 82% (82) | 🏆88% (88) | 🏆88% (88) |
| agent_executability | 50% (50) | 40% (40) | 55% (55) | 🏆80% (80) | 62% (62) |
| verbosity_score | 88% (88) | 🏆90% (90) | 88% (88) | 85% (85) | 85% (85) |
| outcome | 100% (3/3) | 🏆100% (3/3) | 🏆100% (3/3) | 🏆100% (3/3) | 🏆100% (3/3) |
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
- `git-conventions`: a change description carries a visual summary, before and after evidence, a reversibility call, and the checklist.
- `debug`: a tight reproduction loop before any hypothesis, a minimise phase, tagged probes.
- `code-review`: an Intent category, items 37 to 39, checks the diff against the request.
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
run-20261018-120948, 2026-10-18, n=3, claude-sonnet-5-5; ni 2.2.0 runs `ni:full`.

#### plan-easy (home-grown)

| KPI | baseline | openspec | superpowers | ni 2.2.0 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 97% (2 388 tok) | 49% (4 691 tok) | 57% (4 043 tok) | 100% (2 313 tok) | 55% (4 191 tok) |
| cost_usd | 100% ($0.0663) | 39% ($0.1678) | 56% ($0.1187) | 78% ($0.0847) | 39% ($0.1690) |
| duration_s | 88% (21.1 s) | 44% (42.3 s) | 59% (31.2 s) | 100% (18.5 s) | 57% (32.4 s) |
| turns | 100% (4) | 57% (7) | 67% (6) | 57% (7) | 44% (9) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 0% (1) |
| plan_words | 77% (656 words) | 55% (909 words) | 50% (1 006 words) | 100% (504 words) | 96% (527 words) |
| machine_words | 0 words | 251 words | 0 words | 0 words | 151 words |
| human_readability | 88% (88) | 85% (85) | 88% (88) | 88% (88) | 88% (88) |
| agent_executability | 85% (85) | 85% (85) | 90% (90) | 90% (90) | 85% (85) |
| verbosity_score | 85% (85) | 78% (78) | 80% (80) | 88% (88) | 80% (80) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### plan-complex (home-grown)

| KPI | baseline | openspec | superpowers | ni 2.2.0 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 100% (6 412 tok) | 66% (9 739 tok) | 57% (11 229 tok) | 38% (16 869 tok) | 72% (8 900 tok) |
| cost_usd | 100% ($0.1092) | 43% ($0.2563) | 54% ($0.2034) | 30% ($0.3591) | 45% ($0.2443) |
| duration_s | 100% (50.3 s) | 62% (80.9 s) | 65% (76.9 s) | 43% (117.3 s) | 76% (65.9 s) |
| turns | 100% (2) | 18% (11) | 50% (4) | 15% (13) | 18% (11) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 0% (1) |
| plan_words | 68% (2 086 words) | 57% (2 506 words) | 51% (2 775 words) | 69% (2 044 words) | 100% (1 418 words) |
| machine_words | 0 words | 482 words | 0 words | 2 451 words | 555 words |
| human_readability | 90% (90) | 88% (88) | 90% (90) | 88% (88) | 84% (84) |
| agent_executability | 82% (82) | 85% (85) | 92% (92) | 90% (90) | 78% (78) |
| verbosity_score | 80% (80) | 82% (82) | 80% (80) | 72% (72) | 82% (82) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### debug-easy (home-grown)

| KPI | baseline | openspec | superpowers | ni 2.2.0 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 94% (300 tok) | 100% (281 tok) | 40% (704 tok) | 35% (795 tok) | 44% (637 tok) |
| cost_usd | 99% ($0.0308) | 100% ($0.0306) | 43% ($0.0713) | 51% ($0.0598) | 55% ($0.0554) |
| duration_s | 96% (5.7 s) | 100% (5.5 s) | 49% (11.2 s) | 55% (10.0 s) | 72% (7.6 s) |
| turns | 100% (3) | 100% (3) | 43% (7) | 50% (6) | 75% (4) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 60% (60) | 62% (62) | 80% (80) | 85% (85) | 78% (78) |
| agent_executability | 25% (25) | 25% (25) | 40% (40) | 70% (70) | 55% (55) |
| verbosity_score | 90% (90) | 90% (90) | 90% (90) | 85% (85) | 80% (80) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### debug-complex (home-grown)

| KPI | baseline | openspec | superpowers | ni 2.2.0 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 97% (875 tok) | 100% (850 tok) | 69% (1 233 tok) | 64% (1 335 tok) | 62% (1 367 tok) |
| cost_usd | 95% ($0.0479) | 100% ($0.0453) | 53% ($0.0851) | 58% ($0.0779) | 56% ($0.0811) |
| duration_s | 97% (10.4 s) | 100% (10.1 s) | 70% (14.4 s) | 66% (15.3 s) | 66% (15.2 s) |
| turns | 100% (5) | 100% (5) | 71% (7) | 71% (7) | 83% (6) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 72% (72) | 72% (72) | 85% (85) | 88% (88) | 88% (88) |
| agent_executability | 40% (40) | 40% (40) | 55% (55) | 85% (85) | 70% (70) |
| verbosity_score | 88% (88) | 85% (85) | 90% (90) | 85% (85) | 85% (85) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### build-small (home-grown)

| KPI | baseline | openspec | superpowers | ni 2.2.0 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 54% (1 392 tok) | 18% (4 312 tok) | 25% (3 023 tok) | 31% (2 433 tok) | 100% (756 tok) |
| cost_usd | 100% ($0.0551) | 32% ($0.1721) | 42% ($0.1326) | 55% ($0.1004) | 23% ($0.2378) |
| duration_s | 59% (13.3 s) | 18% (43.0 s) | 31% (25.2 s) | 39% (20.2 s) | 100% (7.8 s) |
| turns | 40% (5) | 17% (12) | 20% (10) | 20% (10) | 100% (2) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 72% (72) | 80% (80) | 80% (80) | 85% (85) | 55% (55) |
| agent_executability | 60% (60) | 75% (75) | 78% (78) | 82% (82) | 40% (40) |
| verbosity_score | 85% (85) | 80% (80) | 78% (78) | 82% (82) | 60% (60) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 67% (2/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### ported-debug (superpowers-evals)

| KPI | baseline | openspec | superpowers | ni 2.2.0 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 89% (681 tok) | 100% (605 tok) | 57% (1 059 tok) | 51% (1 188 tok) | 51% (1 198 tok) |
| cost_usd | 98% ($0.0353) | 100% ($0.0345) | 45% ($0.0775) | 48% ($0.0720) | 49% ($0.0703) |
| duration_s | 92% (8.2 s) | 100% (7.5 s) | 61% (12.4 s) | 56% (13.4 s) | 63% (12.0 s) |
| turns | 100% (3) | 100% (3) | 50% (6) | 50% (6) | 75% (4) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 78% (78) | 80% (80) | 82% (82) | 88% (88) | 88% (88) |
| agent_executability | 50% (50) | 40% (40) | 55% (55) | 80% (80) | 62% (62) |
| verbosity_score | 88% (88) | 90% (90) | 88% (88) | 85% (85) | 85% (85) |
| outcome | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) | 100% (3/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |

#### ported-build (superpowers-evals)

| KPI | baseline | openspec | superpowers | ni 2.2.0 | mattpocock |
|---|---|---|---|---|---|
| tokens_total | 100% (1 838 tok) | 49% (3 754 tok) | 52% (3 536 tok) | 58% (3 195 tok) | 58% (3 187 tok) |
| cost_usd | 100% ($0.0586) | 37% ($0.1592) | 50% ($0.1162) | 54% ($0.1090) | 22% ($0.2680) |
| duration_s | 100% (14.8 s) | 36% (41.3 s) | 59% (24.9 s) | 61% (24.2 s) | 42% (35.1 s) |
| turns | 100% (5) | 50% (10) | 45% (11) | 50% (10) | 31% (16) |
| user_turns | 100% (0) | 100% (0) | 100% (0) | 100% (0) | 100% (0) |
| human_readability | 55% (55) | 72% (72) | 72% (72) | 80% (80) | 60% (60) |
| agent_executability | 40% (40) | 50% (50) | 70% (70) | 78% (78) | 55% (55) |
| verbosity_score | 85% (85) | 78% (78) | 82% (82) | 82% (82) | 68% (68) |
| outcome | 100% (3/3) | 67% (2/3) | 100% (3/3) | 100% (3/3) | 67% (2/3) |
| indeterminate | 0/3 | 0/3 | 0/3 | 0/3 | 0/3 |
