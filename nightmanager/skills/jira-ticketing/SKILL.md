---
name: jira-ticketing
use-it-when: "Use before creating or updating any Jira ticket from situation-report hypotheses. Defines eligibility, deduplication and ticket quality rules."
---
Turn actionable hypotheses from a situation report into high-quality Jira tickets that can be implemented by engineers who have not seen the original report.

You are responsible for evaluating, creating and updating tickets using the jira-dc application.

## Eligibility

Only consider hypotheses explicitly listed in the situation report's Hypotheses section.

Use the minimum confidence threshold supplied in the run instructions. If none is supplied, use medium.

Do not change the confidence assigned by the situation-analysis skill.

Only create tickets when there is a concrete change someone could make.

Actionable changes include:
- Application code changes.
- Configuration changes.
- Capacity changes.
- Dependency changes.
- Alerting improvements.
- Dashboard improvements.
- Missing monitoring or logging access.
- Other specific reliability improvements.

Do not create tickets whose only objective is to investigate, inspect or check something.

Suggested checks belong in the situation report, not Jira.

## Draft mode

When the requester asks only to prepare, draft or propose tickets, or the run instructions say so, apply every rule below (eligibility, deduplication, quality) but do not call `create_issue`, `edit_issue` or `add_issue_comment`. Give each draft as the Markdown that passed `ticket-check` (below), with the `jira-find-similar` command and JQL it printed, and report them as drafts, not as Jira actions. Convert with `md2jira` only if the requester wants wiki markup to paste.

## Jira not connected

If the jira-dc app is not available to you, you are in draft mode for the whole run, and you cannot search for existing tickets. Write each draft as above (run `ticket-check` on it if the script is available), and state under *Data gaps* that Jira was not connected, so the drafts were not checked against existing tickets and a person must do that before filing. Never claim a ticket was filed or checked.

## Deduplication

Before creating tickets, retrieve relevant open NightManager tickets for the service. Tickets created by you must carry the label `nightmanager`, because the reporter field shows the person whose connection was used, not NightManager. Run the jira-dc app's `jira-find-similar` (`jira-dc-search` skill) twice, and quote the JQL it prints:

```sh
F="$SKILLS_DIR/jira-dc-search/scripts/jira-find-similar"
"$F" --project <KEY> --label nightmanager                  # NightManager's own open tickets
"$F" --project <KEY> <component> <host> <failure word> ...  # open tickets written by people
```

Use broad words (the component, the host, the mount, the product), not an exact path or error message: reserved characters are dropped and tickets use people's own wording. A related but different ticket is not a duplicate: mention it in the new ticket (`Related: <KEY>-123`).

Also read the *Open items* of your own service memory: a ticket that an operator closed with a decision ("false alarm", "won't fix") is not re-filed. Report the new evidence instead, or comment on a ticket the memory says to comment on.

For each eligible hypothesis:

1. Compare its underlying cause and affected component with existing tickets.
2. Consider whether an existing ticket covers the same problem, even if the wording differs.
3. If it does, update that ticket.
4. Otherwise, create a new ticket.
5. Combine multiple hypotheses that share one underlying cause into a single action.
6. Never create multiple actions for the same underlying problem.
7. Make at most one update per existing ticket during a run.

Do not treat an open ticket as proof that the problem is currently occurring.

If Jira cannot be searched or existing tickets cannot be inspected, report the limitation rather than blindly creating tickets that might be duplicates.

## Ticket quality

Every ticket must be self-contained. Its implementer must not need the original situation report to understand the problem and proposed change.

Use the following structure, written as a Markdown draft: the first line `Summary: …`, then each section as a `## <name>` heading. The Summary becomes the issue's summary field; the remaining sections form the issue description, in order. Text in square brackets is a placeholder; the prose inside the template is guidance for the corresponding section.

```markdown
Summary: [Service] [Component] — [Problem or required change], under 120 characters.

## Problem

[The affected service and component. What is wrong. How the problem manifests itself. The operational or reliability impact, without exaggeration beyond the available evidence.]

## Hypothesis

[The suspected underlying cause, exactly as stated in the situation report, with its original confidence level. Do not present a hypothesis as a confirmed root cause.]

## Evidence

[Exact queries or tool calls from the situation report. Returned values, copied verbatim. Relevant time ranges. Affected hosts and components. Relevant metric names, labels and log fields. Clearly distinguish current evidence from historical context obtained from memories or previous incidents. Do not invent or modify evidence.]

## Contradicting evidence

[Evidence against the hypothesis, copied from the situation report. Do not omit important contradictions simply to make the proposed fix appear more certain.]

## Proposed change

[What should be changed, and where. Why it should address the problem. What reliability improvement it should achieve. Be concrete without inventing implementation details that the evidence does not establish. Prefer fixing underlying causes over individual symptoms.]

## Verification and acceptance criteria

[How the implementer can establish that the change worked. Include a verification query where useful, together with its expected result. Clearly identify proposed verification queries as proposed queries — they have not been executed unless they also appear as executed queries in the original report. Use measurable acceptance criteria wherever possible.]

## Additional context

[Affected components and hosts. Relevant metrics and log fields. Important open questions. Relevant data gaps. Service and analysis time window. Related historical incidents or tickets where useful. Only include information supported by the report, service context or Jira.]
```

## Filing

Use the `jira-dc` app. Before the first filing in a conversation, read `.aurora/apps/jira-dc/app.md`, `tools.md` and the `jira-dc-create-issue` skill. Use the app's named operations (`search_issues`, `get_project`, `create_issue`, `add_issue_comment`, …); `aurora-agent describe jira-dc <operation>` lists their parameters.

```sh
T="$SKILLS_DIR/jira-ticketing/scripts/ticket-check"
M="$SKILLS_DIR/jira-dc-create-issue/scripts/md2jira"
"$T" draft.md --body > body.md                     # template, confidence, placeholders, secrets
"$M" --issue-json --project <KEY> --type <type> --summary "<summary printed by ticket-check>" --label nightmanager < body.md > issue.json
"$T" --issue issue.json                             # summary, description and the nightmanager label
aurora-agent call jira-dc create_issue --body @issue.json --out /tmp/aurora-agent/created.json
```

- Fix every problem `ticket-check` reports before converting; do not file a draft it rejects.
- `md2jira` adds no label by itself: always pass `--label nightmanager`, or deduplication on later runs misses the ticket.
- Find the project key with `list_projects`, and use only an issue type `get_project` lists.
- Descriptions and comments are Jira wiki markup, not Markdown: write Markdown and convert with `md2jira` (`--comment-json` for comments); never hand-convert.
- `create_issue`, `edit_issue`, `add_issue_comment` and `transition_issue` may wait for approval: report a pending approval as pending, not as done.
- A `400` response names the missing or unknown field in `errors`. Fix that field and create once; never retry blindly after an error or timeout, and verify the first attempt really failed before creating again.
- Never include secrets, credentials or tokens in a ticket.
- Report a ticket as created or updated only after Jira confirms it, including the returned issue key.

## Updating existing tickets

When updating an existing ticket:
- Preserve its existing problem description and useful historical information.
- Add materially new evidence when relevant.
- Update confidence only to the value assigned in the current report.
- Record recurrence where supported by repeated observations.
- Avoid duplicate comments or repeated identical evidence.
- Do not close or resolve tickets simply because the problem is no longer visible.
- Do not change ownership, priority or workflow status unless explicitly requested.

## Prioritisation and limits

Order actions by confidence, highest first.

Use the original confidence unchanged.

Respect the maximum number of new tickets per run supplied in the run instructions. If no limit is provided, create no more than three new tickets.

Updates to existing tickets do not count towards the new-ticket limit.

If no hypotheses meet the confidence threshold and actionability requirements, take no Jira action.

## Reporting

For every successful action, record:
- Whether it was created or updated.
- Jira key.
- Summary.
- Confidence.
- Reason for the action.

After Jira confirms a new ticket, add one line to the *Open items* of your own service memory (key, summary, what to do on recurrence), following the service-memory skill.

Report unsuccessful operations as failures. Never claim a ticket was created or updated without confirmation from Jira.