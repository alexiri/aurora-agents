---
name: fixer-memory
use-it-when: "Use before reading or writing any memory: the split between the repos memory people keep and your own notes, what is worth recording, the layout, and how to keep your memory small."
---
Two memories per service. One is the map, and it isn't yours. One is your notebook, and it is.

## The map: `repos-<service>`

Written and kept by the people who run the service, and **locked**: the platform refuses any change or deletion by you. For each repository it says: what it covers, where it lives (GitLab project and numeric id), the default branch and the target branch for merge requests, the test commands and how long they take, the merge request rules (labels, reviewers, squash), and the paths you never touch.

Read it before every ticket. Never try to save to it, even to fix a typo. If something looks wrong or missing, write one line about the difference in your own memory, and mention it in your ticket comment so a person can correct the map.

The service name after `repos-` is the same one NightManager's `service-<service>` memory uses. A `repos-` memory that begins with `EXAMPLE` only shows the format: ignore it.

## The notebook: `fixer-<service>`

Create it the first time you work on a service. Tools: `memory_list`, `memory_get`, `memory_search`, `memory_save` (`key`, `title`, `description`, `content`), `memory_delete`; from the shell, `aurora-agent call aurora memory_get key=…`. A save **replaces the whole text**: read first, change, save the complete result.

Keep it in this order:

```
# The Fixer notes: <service>
Supplements `repos-<service>`. Last tidied: <date>.

## How the repositories really behave     (per repository: a test that needs a flag, a slow step, a trap)
## Open merge requests                     (ticket key, merge request link, state, date, what you're waiting for)
## Declined tickets                        (ticket key, reason, what would change the answer)
## What reviewers asked for                (habits worth following next time)
## Differences from the repos memory       (one line each, until a person has fixed the map)
```

Write down only what a later run would do better for knowing. Don't copy tickets or diffs: the ticket and the merge request hold those. Never write secrets.

## Keeping it small

A memory that only grows costs money and attention in every later run. The budgets are hard:

- **At most 8,000 characters** (about 100 lines). One save is limited to 12,000.
- **Open merge requests:** one line each. A merged or closed one leaves this section only after its Merged or Closed comment is on the ticket (confirmed, not just requested), and then and become one line under "Declined tickets" only if they teach something (closed because it was the wrong fix); otherwise they're deleted.
- **Declined tickets:** at most 15, none older than 60 days, unless a person is still working the ticket.
- **How the repositories really behave:** merge repeats into one line with its last-confirmed date; delete what the repos memory now says; delete what hasn't mattered for 90 days.
- **What reviewers asked for:** at most 10 lines; merge similar ones.

### On every save

1. `memory_get` the current text.
2. Update an existing line before adding one.
3. Apply the budgets to the **whole** text, not just your new lines.
4. Build the final text in a file under `/tmp/aurora-agent/`, check its length with `wc -c`, tidy further if it's over. Never trim by cutting off the end.
5. Save, then `memory_get` again and check it's complete and your change is in it. Report an update as done only after that.

Never drop in a tidy-up: the state of an open merge request; a decline a person asked for; a difference from the repos memory that isn't fixed yet.

### Scheduled tidy-up

If the `Last tidied` date is more than 30 days old, or missing, tidy the notebook at the start of the run before doing anything else, and update the date. If a person asks you to clean up, do the same and tell them what went.
