---
name: post-incident-learning
use-it-when: Use when reviewing resolved Jira tickets or completed incidents to extract confirmed lessons into service memory. Not for open-ticket analysis.
---
Improve your service knowledge by extracting reliable lessons from completed incidents and resolved Jira tickets.

Use this skill when reviewing resolved tickets or when sufficient information becomes available about the outcome of a previously investigated problem.

## Reviewing incidents

Use the Jira application to retrieve relevant resolved tickets. If Jira is not connected, ask for the incident's outcome from whoever requested the review, or stop and say you cannot review it: do not learn from a ticket you cannot read.

Review:
- Original problem description.
- Original hypothesis and confidence.
- Supporting and contradicting evidence.
- Investigation findings.
- Recorded root cause.
- Implemented changes.
- Resolution details.
- Relevant comments and follow-up information.

Do not assume that a ticket's status alone establishes that its proposed fix was implemented successfully.

Look for explicit resolution information and evidence of the actual changes made.

## Confirming outcomes

Distinguish between:

**Confirmed root cause:** The underlying cause was established through documented investigation.

**Implemented remediation:** A specific change was made.

**Verified remediation:** Evidence indicates that the change achieved its intended outcome.

**Unverified resolution:** The ticket was closed or resolved, but there is insufficient information to establish the cause or effectiveness of the fix.

Never claim a remediation worked merely because a Jira ticket is Done.

When a ticket carries a merge request from The Fixer (its comment links it, and the ticket has `fixer-mr`), the change is an **implemented remediation** only once the merge request is merged, and a **verified remediation** only when the ticket's verification query, run after the deployment, shows the expected result. Run it, comment the result on the ticket (see jira-ticketing), and record the outcome in your memory. If it didn't work, say so plainly: that's a lesson (what was tried, what happened) and, with new evidence, the basis for a comment on the ticket for a person to re-triage.

## Learning from incidents

Identify durable information that should improve future investigations:

- Confirmed failure mechanisms.
- Relevant metric signatures.
- Relevant log patterns and fields.
- Affected components and dependencies.
- Conditions that triggered the incident.
- Changes that successfully resolved the problem.
- Useful validation queries.
- Missing observability that complicated diagnosis.
- Previously incorrect assumptions or baselines.

Look for recurring patterns across multiple incidents.

Distinguish recurring symptoms from recurring underlying causes.

## Updating service memory

Use the service-memory skill to add or refine NightManager-maintained knowledge.

For each lesson, retain:
- Affected service and component.
- Description of the incident.
- Confirmed cause, where established.
- Evidence supporting the conclusion.
- Implemented remediation.
- Whether the remediation was verified.
- Relevant Jira key.
- Date and historical context.

Correct previous NightManager knowledge when an incident demonstrates that an earlier hypothesis or assumption was wrong.

Explain in one line why the old entry was superseded; the service-memory tidy-up rules decide how long that line stays.

A reviewed incident should shrink your memory, not grow it. Fold the lesson into a **confirmed failure mode** (signature, trigger, remediation, detection query, ticket key), then delete the *Recent observations* and *Past episodes* lines that told its story. Keep one *Past episodes* line only when the incident has no failure mode to fold into.

Do not modify operator-maintained service memories.

## Baseline implications

Use baseline-management when incident resolution or subsequent healthy operation provides evidence that a baseline should be reconsidered.

Exclude abnormal incident periods from healthy baseline calculations.

Do not learn degraded behaviour as normal simply because it occurred repeatedly.

## Reporting

Summarise important lessons and memory updates.

Clearly distinguish confirmed causes and successful remediations from unresolved questions.

Do not invent missing root causes or remediation outcomes.