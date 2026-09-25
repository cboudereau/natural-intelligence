# Bias checklist

Checklist for the [`bias-analysis`](SKILL.md) skill. Sweep every row against every
option. Skip a row only when the detection question clearly answers no.

## Evidence-pool biases

How the reports got into your pool.

| Bias | Mechanism | Detection question | Correction |
|---|---|---|---|
| Survivorship | Failures leave the pool (scrapped, resold, silent); survivors look like the norm | Do longevity claims come from units still alive? | Look for documented failures; weight them against survivor anecdotes |
| Selection / self-selection | Forums and reviews attract people with problems, or fans | Who bothers to post, and why? | Prefer fleet statistics and surveys with known samples |
| Volume / denominator neglect | Bigger install base, more absolute failure reports | Are counts compared without fleet size? | Divide by fleet size, or downgrade counts to plausible |
| Non-response | Unhappy or happy owners skip the survey unevenly | Is the response rate or sample stated? | Note the gap; cap survey verdicts at moderate |
| Attrition | Long studies lose the worst cases along the way | Did the sample shrink between start and end? | Compare entry and exit populations |
| Publication / vendor | Vendors and sponsored tests publish wins, bury losses | Who paid for or benefits from the source? | Tag as vendor claim; require an independent match |

## Judgement biases

How readers, including you, weigh the pool.

| Bias | Mechanism | Detection question | Correction |
|---|---|---|---|
| Confirmation / brand image | Expected reputation filters which reports feel credible | Does a ranking or statistic contradict the reputation? | Cite the contradicting ranking; state both |
| Availability | Vivid single failures outweigh dull statistics | Does one dramatic anecdote steer the verdict? | Return to base rates |
| Anchoring | Price or spec sets the quality expectation | Is "premium" doing the arguing? | Strip the label; compare measured outcomes |
| Recency | Latest reports dominate; older patterns fade | Would the verdict differ over the full period? | Sample across the whole production run |

## Population-mismatch biases

Whether the pool matches your use case.

| Bias | Mechanism | Detection question | Correction |
|---|---|---|---|
| Usage | Same product, different duty cycles (commercial vs family) | Do reports come from your usage profile? | Segment reports by usage before comparing |
| Maturity | Newer option has less time and fleet to fail | Are the options the same age on the market? | State the hindsight gap; cap the newer option's verdict |
| Geography / language | Local sources over-represent the local brand | Does the source country favour one option? | Mix sources across markets |
| Maintenance | One owner population maintains better than the other | Is upkeep comparable across pools? | Check who actually owns them; test, do not assume |
| Cost asymmetry | Cheap repairs get fixed quietly; expensive ones get posted | Does repair price differ enough to skew reporting? | Note direction; treat repair-report counts as plausible at best |
