# Test 001: HikariCP Connection Acquisition Timeout

## Purpose
Use the exact same synthetic incident with Prompt Versions 1, 2, and 3 to compare the outputs fairly.

## Synthetic incident input
- Endpoint: `GET /api/orders/1042`
- HTTP status: `500 Internal Server Error`
- Time: `14:05`
- Application log: `SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30000ms`
- Additional observation: several requests failed within the same minute.
- Missing evidence: database metrics, pool configuration, query timings, trace data, and deployment history.

## Instructions for the experiment
1. Copy the incident input above.
2. Run it separately with `prompts/v1-basic.md`, `prompts/v2-constrained.md`, and `prompts/v3-investigator.md`.
3. Use the same model for all runs if possible. Do not alter the incident input between runs.
4. Paste each model response under the matching heading below.
5. Do not replace actual results with the illustrative interpretation in the project description.

## Version 1: Actual model response
[PASTE THE ACTUAL RESPONSE HERE]

## Version 2: Actual model response
[PASTE THE ACTUAL RESPONSE HERE]

## Version 3: Actual model response
[PASTE THE ACTUAL RESPONSE HERE]

## Observations after testing
- What did Version 1 do well?
- What did Version 1 miss?
- Which changes in Version 2 improved the answer?
- Which changes in Version 3 improved the answer?
- Did any version invent facts or state a root cause without sufficient evidence?
- What should be changed in a future iteration?

[WRITE OBSERVATIONS AFTER RUNNING THE EXPERIMENT]
