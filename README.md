<p align="center"><img src="assets/logo.svg" alt="ni" width="300"></p>

# ni

**ni** stands for natural intelligence, as opposed to artificial. Say it *nickel* (French: spot on, good enough) or *nice* (UK/US). Either way, it is the human staying in the loop.

ni is a Claude Code plugin, distributed through the [itsaspacestation marketplace](https://github.com/itsaspacestation/claude-marketplace). Its `skills/` folder also works as a plain skills source for Copilot CLI, Cursor, and Gemini CLI.

## Layout
| Path | Purpose |
|---|---|
| `.claude-plugin/plugin.json` | Plugin manifest, name `ni` |
| `agents/` | Subagents with compressed output, spawned as `ni:<agent>` |
| `commands/` | Slash commands, invoked as `/ni:<command>` |
| `scripts/` | Hook scripts behind the terse reply mode |
| `skills/` | The skills, invoked as `ni:<skill>` |

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

For other agents, copy `skills/` into `~/.copilot/`, `~/.cursor/`, or `~/.gemini/`.

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
```

## Release
Users only get an update when `version` in `.claude-plugin/plugin.json` changes.

1. Bump `version` (semver) and commit.
2. `claude plugin validate . --strict`
3. `claude plugin tag .` creates the `ni--v<version>` tag, then push the commit and the tag.

The marketplace entry tracks the default branch, so the marketplace repository needs no change for a release.

## Terse mode
ni injects a terse reply ruleset at session start and reminds Claude every turn, so replies stay short even after context compaction. Adapted from [caveman](https://github.com/juliusbrussee/caveman) (MIT), with two levels only.

| Level | Effect |
|---|---|
| `lite` | Default. No filler, hedging, preamble, or recap. Full sentences kept. |
| `full` | Also drops articles, allows fragments. |
| `off` | Nothing injected. |

Switch with `/ni:terse lite|full|off`. The level persists in `~/.claude/ni/terse`. Persisted text (docs, MR text, comments, commit messages) follows lite rules whatever the level; code and security warnings stay in normal prose.

## Skills
| Skill | Use when |
|---|---|
| `ni:terse` | The terse ruleset itself, for reference or manual invocation |
| `ni:software-engineer` | Implementing, fixing, or refactoring with the plan, test, implement, commit workflow; Rust and .NET build, test, and coverage commands |
| `ni:tdd` | Writing tests first, red-green-refactor |
| `ni:debug` | Any failure or bug, before proposing a fix |
| `ni:plan` | Multi-session work with a durable workspace, design doc, and ADRs |
| `ni:git-conventions` | Any git operation, commit messages, MR or PR descriptions |
| `ni:code-review` | Reviewing a change or answering reviewer comments, GitLab threads via glab included |
| `ni:evidence-based-analysis` | Any claim about the codebase, cited by file and line |
| `ni:skill` | Creating or editing a ni skill, agent, or command |

Run `/ni:help` inside Claude for the same list.

## Commands
User-invoked only; none loads on its own.

| Command | Does |
|---|---|
| `/ni:help` | List the ni skills |
| `/ni:terse` | Set the terse reply level |
| `/ni:review-loop` | Review MRs assigned to me in a /loop, post findings, report the links |

## Agents
Subagent results land in the main context verbatim, so these three return structured one-liners instead of prose. Adapted from caveman's cavecrew (MIT).

| Agent | Use for | Returns |
|---|---|---|
| `ni:investigator` | Where is X defined, what calls Y, map this directory | `path:line - symbol - note` rows |
| `ni:builder` | Surgical edit of 1 or 2 known files | Diff receipt, or `too-big.` / `ambiguous.` |
| `ni:reviewer` | Findings-only review of a diff, branch, or file | `path:L42: severity: problem. fix.` rows |

Rule of thumb: want the result in a third of the tokens, pick ni. Want prose, pick the vanilla agent.
