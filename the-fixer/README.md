# The Fixer

## What it does

Somebody finds the problem. Somebody decides it's worth fixing. The Fixer makes the change.

It takes a Jira ticket a person has marked ready (or a change you describe in the conversation), reads the ticket and every comment on it, works out where the fix belongs in your configuration repositories, makes it, runs your tests, and opens a **draft merge request**. It then comments on the ticket with the link, what it changed, which tests ran, and the query that shows whether the fix worked once it's deployed.

It keeps the change small (under 300 lines and 10 files). It doesn't touch CI configuration, credentials or access rules. If the ticket is wrong, already fixed, too big or needs a decision only a person can make, it changes nothing and says why. That counts as a result.

It can't merge, can't take a draft out of draft, can't run pipelines and can't push to anything but its own branch. A person reads every line.

It talks like someone who's done this a thousand times: calm, short, dry, a bit Australian. Code, commit messages and configuration are written plain.

## Who it's for

Teams that run services from configuration in GitLab (Puppet, Helm charts, Nomad jobs, application config) and track work in Jira, and have more tickets of the "change this setting" kind than hands to do them. It is built to work with **NightManager**, but any ticket a person has marked ready will do.

## What it needs

- **GitLab** (required). The agent reads the repositories your map lists, pushes a branch of its own, and opens and comments on merge requests. It can't merge, delete or run pipelines. But Aurora's gateway sees a push, not which branch it goes to, so **the limits have to be on the GitLab side**:
  - Use a **dedicated account** (a bot user or a project access token), a member only of the projects in your map, with the **Developer** role. Don't connect a person's own token.
  - **Protect every target branch**: allowed to push: no one (or Maintainers), force push off.
  - Mark **CI/CD variables Protected**, and check that pushing a branch on its own deploys nothing (dynamic environments, branch deploy jobs). A push starts a pipeline before anyone has reviewed the change.
  - Optionally add a workspace rule limiting `git_push` and `create_merge_request` to those projects' paths.
- **Jira Data Center** (optional). To find ready tickets and report back. Commenting and labelling ask you first. It never changes a ticket's status. Without Jira it works from tickets you paste in. Connect it with an account that is **not on the triager list and is not the account NightManager uses**: The Fixer ignores its own account's labels and comments, and must not be able to trigger itself.
- **Grafana** (optional, read). It runs the ticket's own query before changing anything, to check the problem is still there.
- **A repository map**, written by you. It lists the ticket project, **who the triagers are**, which **agents may hand it work on a triager's behalf**, and each repository. This is the one thing it can't do for itself. See below.

## Working with NightManager

The two are built to hand work to each other, and agree on how:

1. **NightManager** finds a problem and files a ticket with the label `nightmanager`: the problem, the hypothesis, the evidence, the proposed change (naming the repository and setting when the service notes say where it lives) and a query that shows whether the fix worked.
2. **A triager** reads it and, if it's worth a try, adds the label **`fixer-ready`**. Only a person on the map's triager list does this, and The Fixer checks who added the label. It's the gate.
3. **The Fixer** picks up tickets with `fixer-ready`, makes the change, and opens the draft merge request. It comments on the ticket with the link and adds `fixer-mr` (or `fixer-declined`, with the reason in a comment).
4. **A person** reviews and merges, or doesn't. The Fixer answers the reviewers' comments on the merge request, and when it's merged or closed it says so on the ticket ("Merged: link, time" or "Closed without merging: reason").
5. **NightManager**, in its next runs, sees `fixer-mr` and leaves the ticket alone until that comment. After a merge, it runs the ticket's verification query over a window starting at the merge time and says on the ticket whether the fix worked. After a close, the ticket is an ordinary open ticket again. If The Fixer's schedule is off, ask it to "check my merge requests" after you merge or close one, so the comment gets written (NightManager also reads the merge request itself when it has GitLab access, and flags a `fixer-mr` ticket that has had neither for two weeks).

Neither of them changes a ticket's status. Nobody closes a ticket because the other agent said so. A person who wants another attempt removes the `fixer-mr` or `fixer-declined` label and says why in a comment.

Use the same service name in both: NightManager reads `service-<name>`, The Fixer reads `repos-<name>`.

## Work from other agents

Another agent (your own assistant, say) can ask The Fixer to do a job for you. It counts like you asking directly, but only if the platform shows the request came from an agent listed under **Trusted agents** in the repository map, and the request names a listed triager as the person it's for. Anything else is treated as information, and The Fixer says what to do instead. It answers the agent with the merge request link, or the reason it declined. Delegation never widens a limit: same repositories, same size cap, same forbidden paths, draft merge requests only.

## Setting it up

1. **Create the agent.** Connect GitLab, and Jira and Grafana if you have them.
2. **Write the repository map.** The template includes `repos-example`, which shows what goes in it: the ticket project, the triagers and the trusted agents, and for each repository what it covers, its project id, the branches, the test commands, the merge request rules and the paths to stay out of. Make a new memory called `repos-<your service>` (the same service name as NightManager's `service-<your service>`), fill it in, and **lock it** so the agent can read it but never change it. Delete `repos-example`.
3. **Try it on one ticket.** Mark a small ticket `fixer-ready`, or give it a ticket key in the conversation. Read what it did. Improve the map until it finds the right place without help.
4. **Turn on the schedule.** The template creates a disabled **Pick up ready tickets** schedule (three times a day on weekdays: 09:00, 13:00 and 16:00, in the workspace's time zone). It takes at most two tickets a run. Enable it when you trust it.

## What it does on its own

Only the **Pick up ready tickets** schedule, once you enable it: it works the tickets marked `fixer-ready`, then follows up on review comments and failing pipelines on its own open merge requests. It never messages anyone first.

## Adapting it

- **Limits:** the 300-line and 10-file limits, the forbidden paths (CI configuration and anything the tests or CI execute, such as `Makefile` and `package.json`) and the "two tickets a run" cap are in the instructions and the `change-check` and `ticket-handoff` skills. The map can add forbidden paths per repository, or allow a default one where it's a normal place for a fix. Tighten them freely. If you loosen them, remember that a person has to read the result.
- **Labels:** `fixer-ready`, `fixer-mr` and `fixer-declined` are used by both agents. If you rename them, rename them in both templates' skills.
- **Notes:** its own memory, `fixer-<service>`, is kept under 8,000 characters and tidied by the agent. Read it now and then; fix what's wrong.
- **Updates from newer template versions are offered, never applied automatically.** Memories are never offered again.
