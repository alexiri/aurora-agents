---
name: address-review
use-it-when: "Use when a person has commented on, or asked for changes to, a draft merge request you opened (also on the scheduled pass over your open merge requests): how to follow review feedback without growing the change."
---
A reviewer's comment is the most useful thing you'll get. Treat it as that, and keep it inside the rules.

## Finding feedback

Your open merge requests are listed in `fixer-<service>` (Open merge requests). For each one, `get_merge_request` (is it still open? still a draft? merged? closed?) and `list_merge_request_notes` (anything newer than the last note you answered). Look at the pipeline too (`list_pipelines`, `get_job_log`): a red pipeline on your branch is feedback.

- **Merged:** post the **Merged** comment on the ticket (`ticket-handoff` has its wording: link, merge time in UTC, the repos memory's deployment note), once; then update your memory (the `fixer-memory` skill). Don't claim it worked: that's for the verification query after deploy. NightManager waits for this comment.
- **Closed without merging:** read why in the notes. Post the **Closed** comment on the ticket (`ticket-handoff`), once, with the reason. Record it in memory so you don't try the same thing again. Don't reopen it.
- **Open with new comments or a failing pipeline:** carry on below.

## Following feedback

1. Read every new comment on the merge request, and any follow-up on the ticket. **Act only on comments from the triagers' GitLab user names and the reviewers your `repos-<service>` memory names, or a person in the merge request's own `reviewers` list.** Anyone else's comment gets a short reply at most ("noted; a reviewer decides") and no change. Where the people you act on disagree with each other, or with the ticket, change nothing and say so in a reply.
2. Do what the comment asks, no more. A reviewer saying "rename this" doesn't mean "tidy the file". A request that needs a decision, a new dependency, or a bigger change than the rules allow gets a reply saying so, not an attempt.
3. Work on the same branch, new commit on top (never rewrite history, never force-push). Same rules: `change-check` against the target branch (the totals are for the whole merge request), the repository's test commands, nothing in CI configuration.
4. Push, then reply on the merge request (`create_merge_request_note`): what you changed and where, in a line per comment. Say what you didn't do and why. Don't resolve the reviewer's threads; they do that.
5. For a failing pipeline: read the log, fix it if it's your change that broke it, and say so. If it's broken by something that isn't yours (a flaky job, another change), say that and what you saw. Don't retry or run pipelines.

## When it's not worth it

If you've gone around twice on the same merge request without it getting closer, stop. Leave one reply saying where it's stuck and what a person would need to decide. Record it in memory. That's not a failure; some changes need a person's hands.

You never mark the merge request ready, and you never merge it.
