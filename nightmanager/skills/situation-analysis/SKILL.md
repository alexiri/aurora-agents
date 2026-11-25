---
name: situation-analysis
use-it-when: "Use for any service health investigation: orchestrates the other skills and defines the situation report format."
---
Investigate the health of one service over a specified time window and produce an evidence-based situation report for its on-call operator.

This skill coordinates the investigation. Use the Prometheus, OpenSearch, service-memory and baseline-management skills where relevant.

## Investigation

### 0. Fix the time window

Convert the requested window to absolute UTC timestamps before the first query (the turn context gives local time; the sandbox runs in UTC). Use those RFC 3339 timestamps in every query, never `now-12h`, so that every query in the report can be re-run and gives the same answer. State the conversion in the report.

### 1. Understand the service

Read the service's operator memory (`service-<service>`) and your own (`nightmanager-<service>`), following the service-memory skill. If your own memory's `Last tidied` date is more than 30 days old, tidy it first, as that skill describes.

At the start of the run, after reading your memory: run the watch-for-recurrence skill's clean-up (`grafana-alert expire --managed-by nightmanager`, then fix the registry), so expired tripwires are gone before the investigation starts.

If no operator memory exists for the service, say so as a data gap and work from the dashboards and from what you can discover; do not guess names, data sources or hosts.

Identify:
- Service architecture and components.
- Hosts and instances.
- Dependencies.
- Important metrics and Prometheus labels.
- OpenSearch indices, timestamp fields and log fields.
- Expected behaviour and known issues.
- Available dashboards and runbooks.
- Existing baselines and historical incidents.

Treat operator-provided service context as authoritative guidance. Use NightManager's own memories as historical knowledge.

Identify which monitoring and logging applications are available. Match them to the service context.

Do not assume every data source is available. When the gitlab app is connected and a finding points at a deployment or a configuration limit (worker caps, timeouts, rate limits), read the service's configuration and recent changes there. Read only.

Metrics and logs are queried through the grafana app. Read the app's `grafana-query` skill before the first query and use its script (`$SKILLS_DIR/grafana-query/scripts/grafana-query`). For logs use `grafana-os` (`grafana-opensearch` skill); for peaks, window comparison and "why did this metric jump at time T", the `grafana-prometheus` skill and its `grafana-prom compare`. Record each script command line as the tool call in the report.

### 2. Investigate metrics

Use the prometheus-analysis skill when Prometheus or Grafana metric querying is available.

Check relevant service metrics within the supplied time window.

Look for:
- Traffic and workload changes.
- Error rates.
- Latency.
- Resource utilisation and saturation.
- Capacity constraints.
- Dependency health.
- Sudden changes or unusual patterns.

Compare against the same time window on the previous day when useful. Use validated baselines where available.

### 3. Investigate logs

Use the opensearch-analysis skill when OpenSearch or equivalent log querying is available.

Investigate:
- Error and warning counts over time.
- Most frequent error messages.
- New or unusual log patterns.
- Events that started or stopped at particular times.
- Changes in affected hosts, components or dependencies.

Use bounded queries and aggregations rather than retrieving large amounts of raw log data.

### 4. Correlate findings

Line up metric changes and log events on a common UTC timeline.

Look for:
- Events that occur immediately before degradation.
- Simultaneous changes across components.
- Evidence of dependency failures.
- Resource saturation coinciding with increased latency or errors.
- Changes that explain the observed impact.

Do not assume temporal correlation proves causation.

### 5. Evaluate hypotheses

Use service memories, known issues and previous incidents to guide the investigation.

Identify plausible explanations for the observed behaviour.

For every hypothesis:
- Assign low, medium or high confidence.
- Explain the supporting evidence.
- Explain the contradicting evidence.
- Identify missing evidence or alternative explanations.

A metric anomaly is not automatically a service incident or a root cause.

### 6. Consider persistent knowledge

Use service-memory and baseline-management to identify durable knowledge worth retaining. Be selective: most runs should add nothing, or update one line. A new observation that matches an existing entry refreshes that entry's *last confirmed* date; it is not a new entry.

Do not treat one anomalous observation as a new normal baseline.

### 7. Jira

Use jira-ticketing to evaluate actionable hypotheses against the minimum confidence supplied in the run instructions (default medium, per jira-ticketing).

Do not create tickets for mere suggestions to investigate or check something.

Look at the open tickets for the service before you report. A ticket with the label `fixer-mr` has a fix in review (The Fixer's draft merge request): say so in the report ("fix in review: KEY") instead of treating the problem as unhandled, and follow the jira-ticketing skill's rules for it. A merged and deployed fix gets its verification query run.

## Query rules

These rules are canonical. The data-source skills apply them; do not restate them there.

- Every current observation must be supported by a query executed during this run.
- Include the exact query, tool name and relevant arguments, and returned values.
- Stay within the supplied time window unless a historical comparison or baseline requires otherwise.
- Explicitly identify any queries outside the requested window and give their exact ranges.
- Prefer aggregations, time series and bounded result sets.
- If a query fails or returns no data, adjust it once or twice, then move on.
- Check the unit of every duration or size field on one raw document before interpreting it (for example, compare a response time against size divided by rate).
- Never quote a count taken from printed rows when the output says rows were cut (`... N more rows not shown (M in all)`); use M, and call it a lower bound when M equals the `terms` size you asked for.
- Do not conceal failed queries or missing data.
- Never infer the output of a query you could not execute.

## Report format

Write a Markdown report using the following template. Text in square brackets is a placeholder; the prose inside the template is guidance for the corresponding section of the report.

```markdown
# Situation Report: [Service name]

[Analysis window in UTC]

## Status

One of:

- **OK:** No significant problem is evident in the indicators successfully investigated.
- **DEGRADED:** Evidence indicates reduced service reliability or performance.
- **DOWN:** Evidence indicates the service is unavailable or effectively non-functional.
- **UNKNOWN:** Available evidence is insufficient to assess service health.

The status describes the end of the window. Name any degradation that ended during the window, and any capacity risk, in the explanation (for example "OK — host-3 degraded 03:25–04:10 UTC, recovered; storage 98% full").

One or two sentences explaining the status. Do not claim the entire service is healthy based on incomplete observability.

## Observations

A numbered list of factual findings.

For each observation include:
- What was observed.
- When it happened.
- Affected components, hosts or dependencies, where known.
- Exact query or tool call and arguments.
- Returned values.

Do not put hypotheses in this section.

## Timeline

A UTC timeline of significant metric changes, log events and other observed events, as a table with time and event.

Omit this section if there is only one relevant event.

## Hypotheses

Plausible explanations for the observations. Only include hypotheses supported by the investigation.

For each hypothesis:

**Hypothesis:** Description.

**Confidence:** Low, medium or high.

**Supporting evidence:** Relevant observations.

**Contradicting evidence:** Evidence against the hypothesis.

**Uncertainty:** Important missing evidence or alternative explanations.

## Suggested checks

The most useful checks an operator or engineer could perform next.

These are suggestions, not instructions. Do not claim that a check has been performed when it has not.

## Data gaps

Unavailable data sources, failed or empty queries, missing metrics or log fields, permission problems, and important limitations in the analysis.

Include what was attempted and its outcome.

## Jira actions

Tickets created or updated during the run, including their keys, summaries and actions. Omit if there were no Jira actions.

## Service memory

Durable knowledge added, updated or removed in NightManager's own service memory, and its size before and after if it was tidied. Distinguish validated knowledge from candidate knowledge. Say what happened to the tripwires this run: set, renewed or removed (the watch-for-recurrence skill's registry changes). Omit if there were no memory changes.
```

## Writing style

Use concise, plain English for an operator who knows the service.

Avoid filler, speculation presented as fact and unnecessary explanations of routine operations.