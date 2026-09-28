---
status: accepted
---
# Which GitHub CLI

Addresses: [FR2](../designs/gh-cli-integration.md#fr2), [FR3](../designs/gh-cli-integration.md#fr3), [NFR4](../designs/gh-cli-integration.md#nfr4)

## Problem

The plugin needs one GitHub tool to mirror the `glab` integration. Candidates: `gh` (GitHub's official CLI), `hub` (the older community wrapper), and raw REST/GraphQL through `curl`.

## Options

| Option | Pros | Cons |
|---|---|---|
| `gh` (cli/cli) | Official; active (v2.98.0, pushed 2026-08-20); `--json`/`--jq` on most verbs; `gh api` covers REST and GraphQL for gaps (review threads); auth flow mirrors `glab auth`; command shape (`gh pr list/view/diff/merge/checks`) maps one-to-one onto the plugin's `glab mr` usage | No CLI verb for review threads (GraphQL needed) — same class of gap `glab` has for resolve |
| `hub` (mislav/hub) | `hub api` exists; familiar to old workflows | Last release 2.14.2 (March 2020), dormant; designed as a git proxy, which is the opposite of this design's git-versus-forge separation; no structured `--json` output on porcelain commands |
| Raw REST/GraphQL via `curl` | No new dependency semantics | Reinvents auth and token handling, which the plugin forbids agents to touch; verbose; loses `--jq`; every command becomes an API call |

## Bias sweep (evidence)

Sources: [cli/cli](https://github.com/cli/cli) (vendor, factual release data), [gh-vs-hub.md](https://github.com/cli/cli/blob/trunk/docs/gh-vs-hub.md) (vendor comparison), [hub 2.14.2 release](https://github.com/mislav/hub/releases/tag/v2.14.2) (factual date).

| Bias | Mechanism | Direction | Evidence | Magnitude | Verdict |
|---|---|---|---|---|---|
| Publication / vendor | gh-vs-hub.md is written by the `gh` team | Inflates `gh` | Document hosted in cli/cli | Moderate | Confirmed — its comparative claims are discounted; only its factual design description is used |
| Confirmation ("official = better") | Official status could stand in for capability | Inflates `gh` | Decision re-tested against the capability list the plugin needs (JSON output, GraphQL access, auth status) — `gh` passes on capabilities alone | Weak | Rejected after correction |
| Recency | Active project looks better than dormant | Inflates `gh` | Not a bias here: maintenance is a stated requirement (API drift breaks dormant clients); hub's last release 2020 is factual | — | Rejected |
| Maturity | Newer option has less time to fail | Would deflate `gh` | `gh` is six years old with a large fleet | — | Rejected |

Real advantages, not biases: `gh` alone offers `--json` on porcelain verbs and a maintained `gh api graphql`; both are load-bearing for the thread workflow (FR2).

## Decision

`gh`. It survives the vendor-bias correction on capabilities alone, and it is the only maintained option.

## Consequences

- [`github.md`](../../../skills/code-review/github.md) can mirror [`gitlab.md`](../../../skills/code-review/gitlab.md) almost verb-for-verb; review threads go through `gh api graphql`, matching how [`gitlab.md`](../../../skills/code-review/gitlab.md) already drops to `glab api` for resolve.
- Auth guidance mirrors glab: `gh auth status`, and on failure tell the user to run `gh auth login`; never create or read tokens.
- `hub` is rejected and should not reappear: it is a git proxy, which contradicts [forge-first-boundary](./forge-first-boundary.md).
