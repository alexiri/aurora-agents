---
name: prometheus-analysis
use-it-when: "Use when investigating service metrics in Prometheus or Mimir (usually through Grafana): metric discovery, PromQL construction and interpretation."
---
Investigate service behaviour using Prometheus metrics, through direct Prometheus access or an available Grafana application.

Follow the query rules in the situation-analysis skill: retain the exact PromQL, query tool and arguments, time range and step, and returned values for every finding, and never invent metric values or treat a missing time series as zero activity. 

## Query mechanics

When working through the grafana app, follow the app's `grafana-query` skill and use its `promql` subcommand (`--instant` with `--to`, or `--from/--to/--step`). Put expressions that share a window in one call; it runs them as separate refIds. Add `--wide` for one column per series (host) when comparing instances.

Read the app's `grafana-prometheus` skill before interpreting peaks, caps and capacity: a range query misses peaks shorter than its step, a single sample at a cap is not sustained saturation, and a shared mount appears on every host. Its `grafana-prom compare` does window comparisons (below).

Use the `grafana-query` script (it calls `query_datasources`) for everything, including discovery (below). Some Grafana servers answer the data source proxy operations (`prometheus_query`, `prometheus_series`, `prometheus_labels`, ...) with HTTP 308 or refuse them; the script does not depend on them.

## Metric discovery

Use service memories to identify relevant metrics, labels and targets.

When necessary, discover metrics and labels with instant queries instead of the series/labels endpoints:
- Metric names on a target set: `count by (__name__) ({<selector>, __name__=~"<prefix>.*"})`.
- Values of a label: `count by (<label>) (<metric>{<selector>})`.
- The service's Grafana dashboard panels (`grafana-query panels <dashboard uid>`) show the metric names, labels and expressions the service team uses.

Prioritise metrics associated with:
- Request rate and throughput.
- Error rates.
- Latency and response times.
- CPU, memory and resource utilisation.
- Network and disk I/O.
- Queues, connection pools and capacity.
- Dependencies and application-specific health indicators.

Do not assume metric names or label names. Discover them when they are not documented.

## Query construction

Use efficient PromQL.

Prefer:
- `rate()` or `increase()` for counter metrics.
- `sum by (...)` to aggregate relevant dimensions.
- `topk()` to identify the largest contributors.
- `histogram_quantile()` for latency distributions where histogram buckets are available.
- Range queries to understand changes over time.
- Appropriate rate intervals and query step sizes.

Avoid:
- Returning large numbers of raw time series.
- Unnecessarily grouping by high-cardinality labels.
- Querying unrelated metrics.
- Treating cumulative counter values as event rates.
- Drawing conclusions from a single instantaneous sample when trends matter.

Use the narrowest query that answers the question.

## Investigation

For each important metric:

1. Establish its current behaviour within the requested window.
2. Identify significant increases, decreases or discontinuities.
3. Identify affected instances or components.
4. Compare with relevant historical baselines.
5. Compare with the same time window on the previous day when useful.
6. Investigate whether changes coincide with other metrics or log events.

Consider aggregation levels carefully. A service-wide aggregate can hide a single unhealthy instance.

## Interpretation

Distinguish between:
- Increased workload.
- Increased resource consumption.
- Actual resource saturation.
- Degraded service performance.
- Changes caused by normal workload patterns.

For example, high CPU utilisation alone does not prove CPU saturation is affecting service health.

Consider whether metrics are complete, correctly labelled and representative of the service.

## Historical comparison

When comparing with historical data:
- Use equivalent time windows: `grafana-prom compare --offsets 1d,7d` runs the window and the same window earlier in one call and prints the earlier ranges for disclosure.
- For capacity, compare free space now with free space 7 and 30 days earlier (`offset 7d`, `offset 30d`) and report both rates: one window can hide a recent acceleration.
- Consider workload and operating-mode differences.
- Prefer validated baselines over arbitrary previous samples.
- Disclose any query outside the requested window.
- Do not treat historical behaviour as proof of current service health.

## Output

Provide findings and their evidence to the situation-analysis skill.

When useful, identify candidate baseline improvements for the baseline-management skill. Do not independently change a validated baseline based on a single observation.