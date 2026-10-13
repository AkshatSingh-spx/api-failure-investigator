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
[## Version 1: Basic Prompt Analysis

This prompt is a good baseline for an API Failure Investigator because it provides a real-looking production error, limited diagnostic information, and a request for actionable debugging steps.

### 1. What is the actual problem?

The API endpoint `GET /api/orders/1042` returned HTTP 500 Internal Server Error because the application could not obtain a database connection from its HikariCP connection pool within 30 seconds.

The key error is:

```
SQLTransientConnectionException:
HikariPool-1 - Connection is not available,
request timed out after 30000ms
```

HikariCP is a database connection pool used by Spring Boot applications. Instead of opening a new database connection for every request, it reuses a pool of connections.

When the application needs to query an order, it requests a connection from that pool. If no connection becomes available before the timeout, the operation fails.

### 2. Possible root causes

The log confirms a database connection pool acquisition timeout, but it does not establish why the pool ran out of available connections.

1\. Database connections are exhausted

All available pool connections may be in use by concurrent requests, leaving new requests waiting.

2\. Slow database queries

Queries that take too long can hold connections for extended periods, preventing other requests from using them.

3\. Connections are not released promptly

Long-running transactions, blocked queries, or connection leaks can prevent connections from returning to the pool.

4\. Database or network problems

Database overload, connection failures, or network issues could make connections unavailable or slow to recover.

Because several requests failed within the same minute, a shared resource problem is plausible. However, the available evidence cannot distinguish between these causes.

## 3. Step-by-step debugging guide for a junior developer

1. Confirm the failure in the application logs

   Search for the endpoint, timestamp, exception, and related errors.
   ```
   SQLTransientConnectionException
   HikariPool-1
   GET /api/orders/1042
   ```
   Check whether other endpoints failed at approximately 14:05. If unrelated endpoints also failed, that may point to a shared database connection pool issue rather than a problem specific to order 1042.
2. Inspect HikariCP connection pool metrics

   Check active, idle, and waiting connections, if metrics are available.
   - Active connections at the configured maximum: possible pool exhaustion.
   - Waiting threads increasing: requests are competing for connections.
   - Idle connections available but acquisition still failing: investigate connection validity, timing, and whether the observations align with the failure.
   Metrics are currently unavailable, so this remains a diagnostic step, not a confirmed finding.
3. Investigate database query performance

   Inspect slow-query logs and database activity around 14:05. Look for long-running queries, lock waits, and blocked transactions.

   For the order endpoint, inspect the SQL used to fetch order `1042` and any related data. Do not assume this particular query is responsible merely because its endpoint returned an error.
4. Check for connection leaks and long transactions

   Review code that obtains database connections or starts transactions.

   In Spring Boot, prefer Spring-managed transactions and properly scoped database access. Avoid holding transactions open while making external HTTP calls or performing unrelated lengthy work.

   HikariCP's leak detection can help identify connections held longer than expected, though a warning alone does not prove a leak.
5. Review the connection pool configuration

   Inspect the application's `application.properties` or `application.yml` for settings such as:
   ```
   spring.datasource.hikari.maximum-pool-size=10
   spring.datasource.hikari.connection-timeout=30000
   ```
   These are illustrative values, not the incident's confirmed settings. Compare the pool size with database connection limits, application instance count, workload, and database capacity before changing anything.
6. Check database health and deployment changes

   Examine database availability, connection limits, CPU, memory, network errors, and deployment or configuration changes near 14:05.

   Deployment history is unavailable in this incident, so a recent release cannot currently be blamed or ruled out.
7. Validate the fix under load

   After making an evidence-based change, reproduce the workload in a test environment where possible. Confirm that connection acquisition timeouts stop, requests succeed, and database connections remain within safe limits.

## 4. What should the developer fix first?

Use the evidence to choose the next action rather than randomly increasing the pool size, a popular way to turn one production problem into two.

| Finding                                        | Recommended action                                                                                      |
| ---------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| Pool fully occupied and many waiting requests  | Determine what is holding connections and whether pool sizing matches the workload.                     |
| Long-running SQL queries                       | Examine query plans, indexes, locks, and query execution time.                                          |
| Connections held too long                      | Review transaction boundaries, connection lifecycle, and potential leaks.                               |
| Database at its connection limit or overloaded | Investigate database capacity, competing workloads, and total connections across application instances. |
| Failures began after a deployment              | Compare recent code and configuration changes, then roll back only if evidence supports it.             |

Important: Increasing `maximum-pool-size` or `connection-timeout` without identifying the cause may only delay failures or overload the database further.

## 5. Incident conclusion

CONFIRMED

The order API failed because HikariCP could not provide a database connection within 30 seconds.

LIKELY INVESTIGATION AREA

Database connection pool saturation, slow queries, long-held connections, or database/network issues.

NOT YET CONFIRMED

The exact root cause, the responsible query, the pool configuration, and whether a deployment triggered the incident.

## 6. Limitations of Version 1

Since this is your baseline prompt, these limitations are useful comparison points for later versions.

- No structured root-cause ranking: It lists possibilities but does not rank them by evidence or likelihood.
- No evidence separation: It does not consistently distinguish confirmed facts from hypotheses.
- No diagnostic decision tree: The developer gets a checklist but no clear instructions for what to do based on each result.
- No explicit confidence assessment: It does not explain how strongly the evidence supports a particular hypothesis.
- No immediate mitigation strategy: It focuses on debugging but gives little guidance on reducing ongoing production impact.
- No automated output format: It does not require a consistent incident report that could be parsed or displayed by an API Failure Investigator tool.

Baseline assessment: Version 1 is useful for general troubleshooting and junior-developer education. Its main weakness is that it explains possible causes without providing a sufficiently structured, evidence-driven process for narrowing them down. That gives you a clear target for Version 2: make the investigation more specific, prioritized, and verifiable without pretending unavailable evidence exists.]

## Version 2: Actual model response
[# Spring Boot API Incident Analysis: Version 2

## 1. Incident summary

The `GET /api/orders/1042` endpoint returned HTTP 500 Internal Server Error at 14:05. The application log reports that HikariCP could not provide a database connection within 30 seconds.

The strongest initial lead is database connection acquisition failure, but the available evidence does not establish why the connection was unavailable. The investigation should first determine whether the connection pool was exhausted, connections were being held too long, or the database was unavailable.

## 2. Confirmed facts

- Endpoint: `GET /api/orders/1042`
- HTTP status: `500 Internal Server Error`
- Timestamp: `14:05`
- Exception: `SQLTransientConnectionException`
- Logged message: `HikariPool-1 - Connection is not available, request timed out after 30000ms`
- Several requests failed within the same minute.
- Database metrics, HikariCP configuration, query timings, and deployment history are unavailable.

Interpretation: The application could not obtain a connection from the HikariCP pool within the configured or reported 30-second wait. This does not prove that the database itself was down.

## 3. Ranked hypotheses

### 1. Connection pool exhaustion

Strongest lead

Supporting evidence: Multiple requests failed within the same minute, and HikariCP reported connection acquisition timeouts.

Contradicting evidence: No pool utilization or active/idle connection metrics are available.

Diagnostic check: Inspect HikariCP metrics or logs around 14:05 for active, idle, total, and pending connections.

Confirmation: Active connections reach the pool maximum while requests wait for connections.

### 2. Slow queries or connections held too long

Supporting evidence: Connections may remain occupied while database operations take longer than usual, leaving fewer connections available to other requests.

Contradicting evidence: No query execution timings or connection-hold durations are available.

Diagnostic check: Review database slow-query logs, query durations, and application traces around 14:05.

Confirmation: Long-running queries or extended connection-hold times coincide with the failures.

### 3. Database availability or connectivity problem

Supporting evidence: A database outage, network interruption, or connection failure could reduce the number of usable connections.

Contradicting evidence: No database health, network, or connection-establishment errors are provided.

Diagnostic check: Review database health, connection counts, database logs, and network or driver errors for the incident window.

Confirmation: Database unavailability or failed connection attempts align with the application errors.

### 4. Recent deployment or configuration change

Supporting evidence: A change to pool sizing, request concurrency, transaction handling, or query behavior could trigger connection timeouts.

Contradicting evidence: Deployment and configuration history are unavailable.

Diagnostic check: Compare recent releases and effective HikariCP settings with the last known healthy state.

Confirmation: A relevant change occurred before the failures and its impact is supported by logs or metrics.

These are investigation priorities, not proven root causes. The first two can overlap, and the evidence does not yet distinguish them.

## 4. Diagnostic checks in priority order

| Priority | Read-only check                                                                   | Purpose                                                                            |
| -------- | --------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| 1        | Inspect HikariCP active, idle, total, and pending connection metrics near 14:05.  | Establish whether the pool was exhausted.                                          |
| 2        | Review application logs and stack traces for all failures within the same minute. | Identify whether the failures share a cause or affect multiple endpoints.          |
| 3        | Examine database health, connection limits, and database logs.                    | Determine whether the database or its connections were unavailable or constrained. |
| 4        | Inspect slow-query logs, query durations, and distributed traces.                 | Identify long-running operations and prolonged connection use.                     |
| 5        | Compare recent deployments and effective pool and transaction settings.           | Identify changes that could explain the incident.                                  |

Avoid changing pool sizes, restarting services, or killing database sessions until the initial evidence is captured. Otherwise, the incident may conveniently erase the clues needed to explain it.

## 5. Expected observations and their meaning

| Observation                                                     | Likely interpretation                                                                                       |
| --------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- |
| Active connections at maximum, with pending borrowers           | Pool exhaustion is strongly supported.                                                                      |
| Long-running queries and connections held for extended periods  | Slow database operations or transaction handling may be contributing.                                       |
| Database unavailable or new connections failing                 | Database or connectivity issues become the leading explanation.                                             |
| Pool metrics normal, but connection acquisition still times out | Investigate metric timing, connection lifecycle behavior, pool configuration, and the complete stack trace. |
| Failures begin immediately after a relevant deployment          | Investigate the change, while verifying that the timing and technical evidence support a causal link.       |

## 6. Missing information

The following evidence is needed to narrow the investigation:

- Complete stack trace: To locate where connection acquisition failed.
- Request or correlation ID: To trace the failing request across application and database layers.
- HikariCP metrics and configuration: Including maximum pool size, active connections, idle connections, and pending borrowers.
- Database health and connection limits: To assess availability and capacity.
- Query and transaction timings: To determine whether connections were held too long.
- Deployment and configuration history: To identify relevant changes before the incident.

Credentials, access tokens, and unredacted personal data are not needed.

## 7. Recommended next action

Start by retrieving HikariCP metrics and the complete application stack trace for the 14:05 incident window. Correlate them with database health and query timings.

If the pool was at maximum capacity with waiting requests, investigate why connections were not released quickly enough. If database connectivity was failing, investigate the database and network path instead.

Do not immediately increase the connection pool size. That might mask the symptom or increase database load without fixing the underlying problem.

Current conclusion: The evidence confirms a database connection acquisition timeout in HikariCP. Pool exhaustion is the strongest initial hypothesis, but the root cause remains undetermined until the diagnostic checks are completed.]

## Version 3: Actual model response
[# API Failure Investigator Report

Incident: `GET /api/orders/1042` returning HTTP 500 Investigation mode: Evidence-based, hypothesis-driven Current status:&#x20;

Root cause not established



## A. Incident summary

The API request failed with an HTTP 500 Internal Server Error at 14:05. The application log reports a `SQLTransientConnectionException` indicating that HikariCP could not provide a database connection within 30 seconds.

Several requests failed within the same minute, suggesting a potentially shared issue rather than an isolated request failure. However, the available evidence does not establish whether the underlying cause was database unavailability, connection-pool exhaustion, slow queries, or another condition preventing connections from becoming available.

Leading hypothesis: Database connection-pool exhaustion or connection acquisition delays.

Single most useful next diagnostic check: Inspect HikariCP metrics and application logs around 14:05 to determine whether all connections were active, pending requests were waiting, or database connections were failing.

## B. Evidence table

| Observation                                     | Interpretation                                                      | Limitation                                                                        |
| ----------------------------------------------- | ------------------------------------------------------------------- | --------------------------------------------------------------------------------- |
| Endpoint: `GET /api/orders/1042`                | Identifies the failing API request.                                 | Request ID and trace ID are not provided.                                         |
| HTTP 500                                        | The request resulted in a server-side error response.               | Does not identify the originating failure.                                        |
| Timestamp: 14:05                                | Establishes the reported incident time.                             | Date and timezone are not provided.                                               |
| `SQLTransientConnectionException`               | Indicates a connection-related failure reported by the application. | Does not establish why a connection was unavailable.                              |
| HikariCP connection unavailable after 30,000 ms | A connection could not be obtained within the reported timeout.     | Pool size, active connections, idle connections, and waiting threads are unknown. |
| Several requests failed within the same minute  | Suggests a potentially shared or concurrent failure condition.      | Request volume and the exact failure sequence are unknown.                        |
| Database metrics unavailable                    | Database health and resource utilization cannot be assessed.        | Database saturation or outage cannot be confirmed or excluded.                    |
| HikariCP configuration unavailable              | Pool limits and timeout settings are unknown.                       | Pool misconfiguration cannot be established.                                      |
| Query timings unavailable                       | Query latency and database-side blocking cannot be assessed.        | Slow queries and long-held connections remain possible.                           |
| Deployment history unavailable                  | Recent code or configuration changes cannot be evaluated.           | A deployment-related trigger remains unverified.                                  |

Additional input validation:

- Request ID: Not provided
- Exception stack trace: Not provided
- Database type and topology: Not provided
- Dependency health information: Not provided
- Relevant traces: Not provided

## C. Ranked hypothesis table

The rankings below indicate how well each hypothesis fits the supplied evidence, not statistical probabilities.

| Rank | Hypothesis                                                 | Supporting evidence                                                                                    | Missing or contradicting evidence                                            | Verification step                                                                                                                     |
| ---- | ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| 1    | Connection-pool exhaustion                                 | HikariCP could not supply a connection within 30 seconds; multiple requests failed in the same minute. | No active, idle, maximum-pool-size, or pending-thread metrics.               | Inspect HikariCP metrics around 14:05. Check whether active connections reached the configured maximum and pending threads increased. |
| 2    | Slow queries or long-held connections                      | Connections were unavailable long enough for acquisition requests to time out.                         | No query durations, transaction timings, or connection-hold durations.       | Inspect database slow-query logs and application traces for unusually long queries or transactions.                                   |
| 3    | Database unavailability or connection failures             | The reported exception concerns obtaining a database connection.                                       | No database health checks, connectivity errors, or database logs.            | Inspect database availability and connection-error logs for the incident window.                                                      |
| 4    | Connection leak or connections not being released promptly | Persistent connection unavailability could result from connections remaining checked out.              | No leak-detection warnings, connection-usage timings, or pool recovery data. | Review HikariCP leak-detection warnings and connection checkout/return patterns, if available.                                        |
| 5    | Deployment or configuration change                         | A shared failure affecting several requests can follow a service or configuration change.              | Deployment history and configuration differences are unavailable.            | Compare deployment and configuration-change timestamps with the incident start time.                                                  |

### Classification

- Primary category: Database or connection-pool failure.
- Related categories: Dependency failure; possible query or transaction performance issue.
- Not established: Application logic defect, deployment regression, or database outage.

The exception makes the connection-acquisition path the strongest investigation starting point. It does not prove that HikariCP itself is defective.

## D. First three diagnostic actions

Perform these checks in order. All three are intended to be read-only.

1\. Inspect HikariCP metrics around 14:05

Priority: Highest · Effort: Low to medium

Action: Review historical metrics, if retained, for active connections, idle connections, maximum pool size, and pending threads. Correlate them with application error timestamps.

Expected result and interpretation:

- Active connections at maximum with pending threads: supports pool exhaustion.
- Idle connections available while acquisition still fails: weakens simple pool exhaustion and warrants investigation of connection validity, pool state, and metric timing.
- No historical metrics: this check is inconclusive; proceed to logs and traces.

2\. Examine application logs and traces for the same incident window

Priority: High · Effort: Low

Action: Search for connection-acquisition timeouts, connection creation failures, long-running requests, transaction delays, and HikariCP leak-detection warnings. Correlate requests using trace IDs where available.

Expected result and interpretation:

- Repeated acquisition timeouts and connections remaining checked out: supports pool pressure or connection retention.
- Connection creation or validation errors: supports a database connectivity or connection-health problem.
- Long transactions or slow downstream calls while holding connections: supports delayed connection release.

3\. Correlate database health and query activity

Priority: High · Effort: Medium

Action: Inspect database monitoring, connection counts, resource utilization, database logs, lock waits, and slow-query records around 14:05.

Expected result and interpretation:

- Database connection limits reached or connectivity errors: supports database-side connection pressure or availability issues.
- Long-running queries or lock waits: supports connections being held for extended periods.
- Normal database health and no unusual query activity: weakens these explanations but does not eliminate application-side pool exhaustion.

Safety constraint: Do not restart the service, increase pool size, terminate database sessions, or deploy a fix based solely on this exception. Such actions could erase evidence or make the incident worse.

## E. Missing information needed

The following information would most improve confidence in the diagnosis:

1. Full exception stack trace: Establishes where the connection-acquisition failure surfaced and whether related exceptions preceded it.
2. HikariCP configuration: Maximum pool size, connection timeout, minimum idle connections, and leak-detection settings.
3. Incident-time pool metrics: Active, idle, total, and pending connections.
4. Database telemetry: Availability, connection limits, query latency, lock waits, and resource utilization.
5. Transaction and query timings: Helps distinguish slow database operations from connections that are not being returned.
6. Deployment history: Reveals whether a release or configuration change coincided with the failures.
7. Request correlation details: Request or trace IDs, concurrent request volume, and the exact timestamps of related errors.
8. Recovery pattern: Whether errors stopped spontaneously, persisted, or coincided with a change in traffic or dependency health.

## F. Current conclusion

## Root cause not established

The supplied evidence confirms a database connection-acquisition timeout reported by HikariCP. It does not establish the underlying reason that a connection was unavailable.

Leading hypothesis: Connection-pool exhaustion or prolonged connection occupancy, based on the timeout and multiple failures in the same minute.

Next diagnostic check: Examine historical HikariCP active, idle, maximum-pool-size, and pending-thread metrics at 14:05. This is the most direct way to test whether the pool was saturated before moving to query latency, database health, or connection-retention issues.

Engineering recommendation: Keep the incident classified as a database connection-acquisition failure until additional evidence distinguishes among the competing hypotheses. Avoid treating the exception message as a root-cause diagnosis; it identifies where the request failed to obtain a resource, not necessarily why.]

## Observations after testing
- What did Version 1 do well?
- What did Version 1 miss?
- Which changes in Version 2 improved the answer?
- Which changes in Version 3 improved the answer?
- Did any version invent facts or state a root cause without sufficient evidence?
- What should be changed in a future iteration?

[WRITE OBSERVATIONS AFTER RUNNING THE EXPERIMENT]
