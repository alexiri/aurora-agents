---
name: baseline-management
use-it-when: "Use when establishing, validating or comparing normal service behaviour. Defines the baseline lifecycle (candidate, validated, superseded) and comparison rules."
---
Discover, validate, compare and maintain historical baselines for monitored services.

Baselines help NightManager distinguish normal operational variation from genuinely unusual behaviour.

Use the Prometheus-analysis and service-memory skills where relevant.

## Baseline discovery

Identify useful service metrics from operator-maintained memories and previous investigations.

Examples include:
- Request throughput.
- Error rates.
- Latency distributions.
- CPU and memory utilisation.
- Network and disk activity.
- Queue depth.
- Connection pool utilisation.
- Dependency response times.

Use historical metric queries to establish normal behaviour. `grafana-prom compare` (grafana-prometheus skill) gives min, p50, p95, max and mean of one expression over a window and the same window 1 and 7 days earlier (`--offsets 1d,7d`, or more offsets for a longer history) in one call; use it for candidate baselines and for comparisons, and disclose the earlier windows it prints.

Do not assume that the values in operator-provided service context are validated statistical baselines. Treat them as operational guidance unless their provenance is clear.

## Selecting historical periods

Choose representative periods when the service was operating normally.

Where possible:
- Exclude known incidents and degraded periods.
- Exclude unusual maintenance and deployment windows.
- Consider daily and weekly patterns.
- Account for workload and traffic differences.
- Consider different operating modes.
- Use sufficient historical coverage to avoid baselines based on isolated observations.

If there is insufficient information to identify healthy periods, retain a candidate baseline rather than declaring it validated.

## Calculating baselines

Choose statistics appropriate to each metric.

Examples:
- Average and standard deviation for suitable stable measurements.
- Median and percentiles for skewed distributions.
- Minimum and maximum where useful.
- Rates and distributions for counters and histograms.
- Time-of-day and day-of-week comparisons for periodic workloads.

For latency, consider p50, p95 and p99 where the underlying metric supports them.

Avoid using averages alone when they conceal important tail behaviour.

Segment baselines by workload, environment or operating mode when these conditions materially affect normal behaviour.

## Comparing current behaviour

When analysing a service:
1. Retrieve the relevant validated baseline.
2. Compare current measurements against equivalent historical conditions (`grafana-prom compare`: a `p95 now / median p95 before` near 1x is normal for that time of day; 2x or more is a deviation to explain).
3. Identify meaningful deviations.
4. Consider whether workload differences explain the deviation.
5. Identify whether the deviation corresponds to actual service degradation.

A deviation from baseline is an observation, not automatically an incident.

A high resource-utilisation value is not necessarily abnormal if it is expected under the current workload.

## Baseline lifecycle

Distinguish three states:

**Candidate:** Newly calculated or proposed, but not sufficiently validated.

**Validated:** Supported by sufficient representative, healthy historical data.

**Superseded:** No longer representative of the current service configuration or operating conditions.

Do not silently replace a validated baseline with a candidate.

When proposing a new baseline:
- Explain why the existing baseline is insufficient.
- Identify the historical observation period.
- Provide the query and source.
- State the calculated statistics.
- Explain the operating conditions.
- Assign confidence.
- Explain any limitations.

## Baseline maintenance

Review existing baselines when:
- Service architecture changes.
- Workload characteristics change.
- Deployments materially alter performance.
- Repeated healthy observations differ from the existing baseline.
- A baseline becomes too old to be representative.
- A previously unknown periodic pattern is identified.

Do not automatically learn from the most recent observations. New observations may reflect a degradation that should remain detectable.

## Recording

Use the service-memory skill to record candidate, validated and superseded baselines in NightManager's own service memory.

Preserve:
- Metric and unit.
- Statistic and value.
- Relevant labels or segmentation.
- Observation period.
- Operating conditions.
- Source and exact query.
- For a log count (a baseline can be one): the Lucene query, the index, the window length, and the typical value and spread over the same periods as for metric baselines.
- Confidence.
- Last validation date.
- Status.

Keep baselines compact: one line per metric and segment, in this shape, with the exact query stored once:

```
- apache busy workers (fleet, weekday): p50 70, p95 135, max 140 | 2026-11-16..22, 1m | validated 2026-11-23 | q: max(apache_workers{…,state="busy"})
```

Only the current validated value and its period stay. When a baseline is superseded, replace the line (one note of what replaced it and why, expiring under the service-memory tidy-up rules) rather than stacking versions. Candidates that have not been validated or refreshed for 30 days are deleted.

Never invent a baseline value, observation period or validation result.

## Reporting

In situation reports, distinguish current values, historical baselines and calculated deviations.

Report when a comparison is unavailable or a baseline has insufficient confidence.

Never present an unvalidated baseline as established normal behaviour.