---
name: opensearch-analysis
use-it-when: "Use when investigating service logs in OpenSearch (usually through Grafana): field discovery, bounded aggregations, raw samples and interpretation."
---
Investigate service logs using OpenSearch or an available logging application.

Identify errors, warnings, unusual patterns and events that help explain observed service behaviour.

Follow the query rules in the situation-analysis skill: retain the exact query, tool invocation and arguments, index or data stream, timestamp range, and returned values for every finding, and never invent log messages, fields, hosts or event counts. Copy relevant queries and returned values into the situation report.

## Query mechanics

When working through the grafana app, use the app's `grafana-opensearch` skill and its `grafana-os` script: counts, sums, percentiles, `terms` and `date_histogram` from flags (`--metric`, `--terms`, `--hist`, `--wide`), and documents with `--raw N --columns a,b,c`. Add `--print-body` and put the printed body in the report as the query that was run. Write a body for `grafana-query run` only for what the flags cannot express (several refIds, `filters` buckets). The plugin's quirks (no `logs` metric, `percentiles` needing `--hist`, `cardinality` on IPs) are listed in that skill; do not work around them by hand.

## Discovery

Read service memories for:
- Index patterns and data streams.
- Timestamp fields.
- Log severity fields.
- Service, host and component identifiers.
- Application-specific fields.
- Known error messages and patterns.

When necessary, discover actual field names. Index mappings are not reachable through Grafana. Read the service dashboard's log panels (`grafana-query panels <dashboard uid>`: `query`, `metrics[].field`, `bucketAggs[].field`, `timeField`) and fetch one to three documents (`grafana-os ... --raw 3`). Check the unit of every duration and size field on that sample before using it (situation-analysis query rules).

Do not assume that different services use the same mappings or naming conventions.

## Query construction

Use bounded OpenSearch queries.

Prefer:
- Time-range filters on the correct timestamp field.
- Filters for service, component and host.
- `date_histogram` aggregations for event counts over time.
- `terms` aggregations for frequent error messages, hosts and components.
- Count queries for aggregate volumes.
- Pattern-analysis tools where available.
- Small, targeted samples of documents when necessary to understand an unfamiliar log structure.

Avoid:
- Unbounded searches.
- Returning thousands of raw log documents.
- Large raw-document samples (`--raw` returns every field; keep N at 50 or less and select fields with `--columns`).
- Aggregating across unrelated services.
- Using the wrong timestamp or severity field.

Always include the requested time window in relevant queries.

## Investigation

Investigate:

1. Error and warning volumes over time.
2. Most frequent error messages and patterns.
3. New or unusual messages.
4. Changes in log volume.
5. Events that started or stopped at particular times.
6. Hosts and components generating errors.
7. Relevant events around metric anomalies.

Compare log events with the metric timeline.

Where possible, identify the first occurrence of a new error pattern and whether it coincides with degraded service behaviour.

## Interpretation

Distinguish between:
- Increased logging volume and an increased failure rate.
- Repeated messages and independent failures.
- Expected warnings and abnormal warnings.
- Application errors and downstream dependency errors.
- A log event correlated with an incident and a proven cause.

Log counts alone do not necessarily represent request error rates. Consider sampling, duplicate messages and logging coverage.

Do not claim a log pattern is new unless you have suitable historical evidence.

## Output

Provide relevant findings and evidence to the situation-analysis skill.

Identify useful new log fields, error signatures and recurring patterns for service-memory and post-incident learning.

Do not promote an unverified log pattern into an established failure mode.