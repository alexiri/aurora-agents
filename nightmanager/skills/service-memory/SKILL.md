---
name: service-memory
use-it-when: "Use before reading or writing any service memory. Defines the split between operator-maintained context and your own knowledge, what is worth recording, and how to keep your memory small (budgets and the tidy-up procedure)."
---
Maintain an accurate, useful and compact knowledge base for each monitored service.

Memories guide investigations and hold durable knowledge discovered during investigations and incident reviews. They are not a log of what happened: situation reports and Jira hold that.

## Memory ownership

There are two kinds of service memory, told apart by key.

### Operator memory: `service-<service>`

Written and kept by the service team, and **locked**: the platform refuses any change or deletion by you. It typically holds:
- purpose, ownership, architecture and components;
- hosts and infrastructure, dependencies;
- metric names, labels, data source names and ids, queries;
- log indices and fields;
- dashboards, runbooks, the Jira project;
- normal behaviour and known issues.

Treat it as authoritative context. Never try to save to it, even to fix a typo.

If something in it looks outdated, contradictory or incomplete, record the discrepancy in your own memory (one line, with the evidence) and mention it in the situation report. Do not "fix" your own memory to agree with it silently.

A `service-` memory whose text begins with `EXAMPLE` only shows the format. It describes no real service: ignore it.

### Your own memory: `nightmanager-<service>`

Create it on the first run for a service, if it is missing. Use it for:
- historical behaviour and recurring patterns;
- validated baselines (candidates too, marked as such);
- confirmed failure modes and their signatures;
- tickets you filed and what became of them;
- operational lessons.

`nightmanager-<service>-baselines` holds baselines once they no longer fit in the main memory.

Tools: `memory_list`, `memory_get`, `memory_search`, `memory_save` (`key`, `title`, `description`, `content`) and `memory_delete`; from the shell, `aurora-agent call aurora memory_get key=…`. A save **replaces the whole content** of the key, so always read first, change the text, and save the complete result. The files under `memory/` are a read-only projection of the same notes: reading them is fine, but save through `memory_save` so the key is explicit.

## Reading memories

Before investigating a service:

1. Read its operator memory.
2. Read your own memory for it.
3. Pick out the architecture, dependencies, metrics, baselines and known failure modes that matter for this window.

Use history to decide what to look at; establish current conditions with current queries. Do not assume an old incident is recurring because the symptoms look alike.

## What is worth recording

Record a fact only when a later run would investigate better because of it. Ask: *would I want this at the start of the next run, and is it still true next month?* If not, it belongs in the report or the ticket, not in memory.

Distinguish these categories, and label each entry with its category and confidence:

- **Established facts**: verified information about architecture or behaviour ("`/mnt/data` is one shared mount on all hosts"; "`response_time` is in microseconds").
- **Validated baselines**: expected behaviour from representative, healthy data, with period, statistic, query and last-validated date.
- **Confirmed failure modes**: a mechanism supported by incident evidence or a documented root cause: signature (metrics and log patterns), trigger, remediation, the query that detects it.
- **Recent observations**: dated, short, not yet known to matter. These expire (see below).
- **Open items**: tickets you filed or follow, with key and state.

**Hypotheses stay out of memory.** They belong in reports and Jira. Promote one only after further evidence confirms it, and then record it as a confirmed failure mode.

Every entry should carry what a reader needs to trust it: source or query, period observed, date first seen, date last confirmed, ticket keys, and status (established, provisional, superseded).

Never store secrets, credentials or tokens.

## Layout

Keep the main memory in this order, so that tidying is mechanical:

```
# NightManager knowledge: <service>
Supplements `service-<service>`. Last tidied: <date>.

## Established facts
## Validated baselines            (or: see nightmanager-<service>-baselines)
## Confirmed failure modes
## Open items                     (tickets: key, summary, state, what to do on recurrence)
## Recent observations            (dated, newest first)
## Past episodes                  (one line each)
## Discrepancies with the operator memory
```

## Keeping memory small

An unbounded memory costs money and attention in every later run, and eventually stops fitting: a single save is limited to 12,000 characters. Treat these budgets as hard:

- **Main memory: at most 8,000 characters** (about 100 lines). This leaves room to add before the next tidy-up.
- **Baselines memory: at most 8,000 characters.**
- **Recent observations: at most 15 entries, none older than 14 days.**
- **Past episodes: one line each, at most 20.**

### On every save

Never append blindly. Do this each time you save:

1. `memory_get` the current text.
2. Make your change in the whole text: update an existing entry before adding a new one (check for it first; also with `memory_search`).
3. Run the tidy-up rules below over the whole text, not just your new lines.
4. Build the final text in a file (under `/tmp/aurora-agent/`) and check its length with `wc -c`. If it is over budget, tidy further, or move baselines to the `-baselines` memory. Never trim by cutting off the end.
5. Save, then `memory_get` again and confirm the text is complete and your change is in it. Report the memory update as done only after that.

### Tidy-up rules

1. **Merge repeats.** The same fact seen again is one line: keep the first-seen date, the *last-confirmed* date and the latest value. Drop the intermediate re-checks.
2. **Trends, not samples.** A series of readings (free space, error counts over days) becomes the first and last value, the rate, the period and the query that reproduces it.
3. **Expire recent observations after 14 days.** Then do one of three things: (a) it turned out to matter and recurred or has a ticket → fold it into a confirmed failure mode or a baseline; (b) it was a real episode → one line in *Past episodes*: date, what, cause if confirmed, ticket key; (c) it was benign, a one-off or noise (a single-sample peak, scanner probes, one-time bursts) → delete it.
4. **Expire past episodes** that have not recurred in 90 days and have no open ticket, unless they are the only evidence for a failure mode.
5. **Superseded entries** (an old baseline, a disproven assumption): keep one line saying what replaced it and why, for 30 days; then delete the line unless a ticket or failure mode refers to it.
6. **Closed tickets**: remove the story, keep the key and the outcome *only if it tells a later run what to do* ("closed as a false alarm; do not re-file", "comment on it on recurrence"). Otherwise drop it.
7. **Examples, not inventories.** Three example paths, hosts or client addresses are enough; do not keep lists.
8. **Never keep** run state ("re-check at 22:31"), who was told what, draft ticket text, query output beyond the few values that define a baseline or a signature, anything the report or Jira already holds, or anything that is just a copy of the operator memory.
9. **Never drop in a tidy-up**: the key and state of an open ticket; an operator decision that affects what you do (a ticket closed as a false alarm, a limit someone set deliberately); validated baselines; the signature and detection query of a confirmed failure mode; a discrepancy with the operator memory that is still unresolved.

### Scheduled tidy-up

Besides tidying on each save:

- **At the start of a run, if the memory's `Last tidied` date is more than 30 days old** (or missing), do a full tidy-up pass before the investigation, even if you have nothing new to add, and update the date.
- If a person asks you to clean up your memory, do the same pass and tell them what you removed.

Deleting a whole memory (`memory_delete`) is for your own notes only, when the service is gone or the memory is entirely obsolete, and only when an operator has told you so. Never delete the operator memory (the platform refuses anyway).

## Updating existing knowledge

- Update an entry when new evidence refines it; do not duplicate it.
- When a new observation disagrees with an established baseline, find out whether the service changed, the workload differs or the baseline is stale. Record the conflict (one line) instead of silently choosing one side.
- Mark an entry superseded when it no longer applies, and say what replaced it.
- A baseline is replaced by a candidate only through the `baseline-management` lifecycle.

## Reporting

At the end of a run, state in the report's *Service memory* section what you added, updated, removed (and why), left unchanged or could not validate, with the memory's size before and after when you tidied it (for example "tidied: 9,400 → 6,100 characters; removed 4 expired observations, merged 3 repeats"). Only report a change as made when it has been saved and read back.
