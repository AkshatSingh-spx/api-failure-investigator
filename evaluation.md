# Evaluation: Prompt Iterations

## Evaluation method
Run the same synthetic incident with all three prompt versions, ideally using the same AI model and settings. Score each output from 1 to 5 against the same criteria.

## Rubric
| Criterion | Weight | What to check |
|---|---:|---|
| Evidence accuracy | 30% | Does it distinguish observations from assumptions and avoid invented facts? |
| Diagnostic usefulness | 25% | Are next steps specific and actionable? |
| Uncertainty handling | 20% | Does it clearly identify unknowns and avoid unsupported certainty? |
| Output structure | 15% | Is the report organized and consistent? |
| Safety | 10% | Does it prefer safe checks and protect sensitive information? |

Score meaning: 1 = poor, 2 = weak, 3 = acceptable, 4 = strong, 5 = excellent.

Weighted score out of 5 = sum of (criterion score × criterion weight) / 100.

## Results
Fill this table only after testing. Do not invent scores.

| Version | Evidence accuracy /5 | Diagnostic usefulness /5 | Uncertainty handling /5 | Output structure /5 | Safety /5 | Weighted total /5 |
|---|---:|---:|---:|---:|---:|---:|
| Version 1 | 30% | TBD | TBD | TBD | TBD | TBD |
| Version 2 | TBD | TBD | TBD | TBD | TBD | TBD |
| Version 3 | TBD | TBD | TBD | TBD | TBD | TBD |

## Findings
After testing, record concrete observations here:
- Version 1: 
- Version 2:
- Version 3:
- Most useful improvement:
- Remaining weakness:
- Next iteration to try:

## Limitations
- One synthetic incident is not enough to establish general performance.
- Outputs can vary between runs and models.
- Scores are human judgments, not a formal accuracy benchmark.
- A prompt is a debugging aid, not proof of root cause. Human review is required before operational changes.
