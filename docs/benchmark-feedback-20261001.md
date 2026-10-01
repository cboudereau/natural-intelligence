# Benchmark feedback — 2026-10-01

Source: ni-bench run-20261001-190237 (4 arms: baseline, openspec, superpowers, ni;
7 scenarios × n=3; blind sonnet judge, rubric v2 with human_readability and
agent_executability; explicit workflow invocation on plan, debug, and build families;
skill invocation verified in session logs). ni scored 21/21 outcomes — the findings
below are the measured gaps, each with the judge-note evidence and the skill change
made in response.

## 1. No effort scaling in `plan` (verbosity 68–72, lowest on both plan scenarios)

Evidence: judge notes on plan-easy — "heavy process scaffolding (PREFLIGHT, ADR,
CLAUDE…)", "heavier than a simple task needs, with an ADR, a mermaid…". 11 subject
turns vs baseline 3; 2.7× baseline tokens on plan-complex for tied human_readability
(90). The full workspace ceremony ran on a one-requirement task.

Change: `skills/plan/SKILL.md` gains an **Effort scaling** section — one plan document
for single-requirement, single-session tasks; ADR only for a genuinely hard-to-reverse
decision; full workspace only beyond one session or ~5 tasks.

## 2. Process trace invisible in summaries (debug agent_executability 35–62)

Evidence: every ni debug/build judge note — "the transcript shows no tool calls, so
reproduction, root-cause investigation and the test re-run can't be verified". The
root cause was correct in 9/9 debug trials; the proof was stripped by terse replies.
Competing arms that narrate their process scored up to +27 points on the same work.

Change: `skills/debug/SKILL.md` gains an **Evidence block** — the final summary closes
with three fixed lines (reproduced, root cause before fix, tests red→green), whatever
the terse level.

## 3. Red-first skipped silently (ported-build human_readability 62; build-small)

Evidence: judge notes — "admits the tests were never run red and that the planning
skill was skipped" (ported-build t02), "the implementation was written before the
tests" (build-small t01). The honest admission scored better than silence; neither is
as good as doing it.

Change: `skills/tdd/SKILL.md` Discipline gains **Record the red run** (keep the failing
command and output line, quote it in the report) and **Skipping is stated, never
silent**.

## 4. Thin final reports (ported-build t01 "gives no persistence details")

Evidence: correct work lost points when the summary did not account for every
requirement in the brief.

Change: `skills/software-engineer/SKILL.md` gains a **Final report** step — one
disposition line per requirement (what, where, verified how), the red→green line, and
explicit mention of any skipped step.

## Not changed, on purpose

Plan substance (both hard decisions made with alternatives and constraint-tied
reasons, human_readability 88–90), root-cause discipline (correct cause 9/9, quick-fix
pressure rejected 3/3), and outcome reliability (21/21) measured at or above every
other arm — the gaps were economy and evidence visibility, not method.

## Benchmark caveats

The judge reads final messages and markdown artifacts only, so process scores partly
grade self-reporting; the benchmark is run by ni's author (disclosed in its analysis);
skills required explicit invocation in headless one-shot prompts — why they do not
trigger autonomously there is an open analysis item on the benchmark side.
