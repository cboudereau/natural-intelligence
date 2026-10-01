# Benchmark feedback — 2026-10-01, round 2

Source: ni-bench run-20261001-211636 (ni 1.7.0 — the A/B readout of round 1, same
harness and prompts as the 1.5.0 run). Round 1 held: outcomes 21/21, debug
agent_executability moved from last place to first (35→62, 55→82, 55→72), plan-easy
tokens −61% with best plan_words and verbosity. The judge notes now expose five
residuals.

## 1. Red/green is claimed, never quoted (build agent_executability 72/62 vs superpowers 82/70)

Evidence: "says tests were written first… then pass" (build-small t01, ae 55); "no raw
red/green output in the transcript" (t02); "reportedly written before the code, but no
commands or output to show it" (ported-build t01). The round-1 rule said record the
red run; agents paraphrased it.

Change: `skills/tdd/SKILL.md` — the report quotes the command and decisive output
lines verbatim, with a concrete example; a paraphrase is a claim, the quote is
evidence. This is reviewer-grade practice independent of any benchmark judge.

## 2. Red-first still skipped when plan drives a build (1 of 3 ported-build trials)

Evidence: "admits tests were not run red-first" (ported-build t03). The stated-skip
rule worked; the skip itself persists on the plan-driven path.

Change: `skills/plan/SKILL.md` — when this skill drives an implementation, the tdd
red run is a gate: implementation starts after the failing run is captured.

## 3. ADR ceremony leaks into small builds

Evidence: "the ADR is extra process for a small task" (ported-build t03).

Change: `skills/plan/SKILL.md` — the effort-scaling ADR threshold applies to builds
too: inline decision bullets unless the decision is genuinely hard to reverse.

## 4. Ratification questions in delegated contexts (plan-complex user_turns 2, $0.731 worst trial)

Evidence: ni asked the requester mid-plan although the brief delegates ("your call,
decide and continue"); 13–19 turns vs baseline 2, cost up to 5× baseline on the
affected trials.

Change: `skills/plan/SKILL.md` — delegated or one-shot context: decide, record the
decision with its reason, continue; ask only for a genuinely blocking externality.

## 5. Effort scaling deleted the machine layer entirely (machine_words 2 802 → 0 on plan-complex)

Evidence: the run's two-audience split shows no machine-facing artifact at all on
single-session work — the judged scores held, but the resumability product (crash,
rate-limit, compaction recovery from disk) silently disappeared with the cost.
Overcorrection of round 1.

Change: `skills/plan/SKILL.md` — the single document always keeps a minimal machine
section (task checkboxes with verify commands and a resume line, a few dozen words);
the full TASKS.md returns at multi-session scope. A plan an agent cannot resume from
is a chat message, not a plan.

## 6. Completeness: negative requirements (ported-debug −3 to −8 human_readability)

Evidence: "never says that unknown SKUs still fail loudly" (ported-debug t02).

Change: `skills/software-engineer/SKILL.md` — the final report names preserved
behaviours (negative requirements) with their own disposition line.

## Guardrail for this round

The benchmark's own analysis discloses a teaching-to-the-test risk: round 1 was
written knowing the rubric. Every round-2 change is defensible without the judge:
quoted test output is what any reviewer asks for; deciding under delegation is the
brief's own instruction; the resume checklist serves the crash case, which the judge
does not score at all. No change whose only effect is a judge score was made.

## Benchmark-side counterparts (tracked in the ni-bench repo)

Stage-2 kill-and-resume KPI (two-audience-quality ADR) — now the priority, since the
machine layer became conditional; n=5 on build scenarios (openspec's 1/3 → 3/3 swing
shows n=3 noise); judge gets the tool log so process scores stop depending on
self-reporting.
