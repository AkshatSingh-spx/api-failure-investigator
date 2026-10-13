# API Failure Investigator
A reusable prompt engineering system for investigating Spring Boot API failures using evidence-based analysis.

## Problem
When a backend API returns an error, developers may receive incomplete logs and need to decide what to investigate first. A generic AI prompt can produce broad advice or state an unverified cause as fact.

## Goal
Turn supplied API failure evidence into a structured, prioritized debugging plan while clearly separating facts, hypotheses, and unknowns.

## What is included
- `prompts/v1-basic.md`: baseline prompt.
- `prompts/v2-constrained.md`: adds technical context, evidence rules, and a fixed output structure.
- `prompts/v3-investigator.md`: adds hypothesis verification, diagnostic prioritization, and safety checks.
- `tests/incident-001.md`: shared synthetic test case and a place to record the actual outputs from each version.
- `evaluation.md`: scoring rubric and results table.

## How to use
1. Copy the incident in `tests/incident-001.md`.
2. Run it separately with each prompt version in the same AI model, if possible.
3. Paste each actual model response into the test file.
4. Score the outputs in `evaluation.md` using the same rubric.
5. Record what improved, what did not, and any limitations.

## Important
The incident is synthetic. Example interpretations are not verified production findings. Do not claim a root cause unless the supplied evidence supports it. Do not include credentials, access tokens, personal data, or confidential company logs.

## Scope and limitations
This is a prompt-engineering prototype, not a production incident-response system. AI output must be reviewed by a qualified developer before operational changes are made.
