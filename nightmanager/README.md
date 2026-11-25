# NightManager

## What it does

NightManager is a Site Reliability Engineering agent that looks after one service at a time while you are away:

- **Investigates** the service over a time window (the last 12 hours by default). It queries Prometheus metrics and OpenSearch logs through Grafana, compares them with the same window a day and a week earlier, and writes a **situation report**: status (OK, DEGRADED, DOWN or UNKNOWN), numbered observations with the exact queries and returned values, a timeline, hypotheses with a confidence and the evidence for and against, suggested checks, and the data it could not get.
- **Files Jira tickets** for hypotheses that are specific enough to act on and confident enough (medium by default). It first looks for an existing ticket and updates that instead of filing a duplicate. Every ticket it files carries the label `nightmanager`.
- **Keeps notes** about each service: baselines of normal behaviour, confirmed failure modes, what became of its tickets. It keeps those notes small by merging repeats and dropping what has stopped mattering.

It reads; it does not fix. It cannot change dashboards, alerts or silences, and it makes no change to your services. It never messages anyone on its own. Creating or commenting on a Jira ticket waits for your approval, because the Jira connection is set to ask. **Jira is optional**: without it the agent only reports, and puts ticket drafts in the report.

## Who it's for

Teams who run a service with Prometheus metrics and OpenSearch logs in Grafana and track their work in Jira, and who want a first look at overnight or weekend trouble before an engineer starts: what happened, how sure the agent is, and what a ticket should say.

## Working with The Fixer

NightManager has a teammate: **The Fixer** (another template in this repository). NightManager files tickets with the label `nightmanager`; a triager adds `fixer-ready` to the ones worth a try; The Fixer opens a draft merge request and labels the ticket `fixer-mr`; a person reviews it; The Fixer says on the ticket when it's merged (or closed); and NightManager then runs the ticket's verification query from the merge time and says whether the fix worked. Neither agent changes a ticket's status, and only a person adds `fixer-ready`. Use the same service name in both: NightManager reads `service-<name>`, The Fixer reads `repos-<name>`. Name the repositories where the configuration lives in your `service-<name>` memory so tickets can say where a change belongs. NightManager works fine without The Fixer.

## What it needs

- **Grafana** (read): the agent runs its queries as the account you give it. Give it a Viewer service account that can see the data sources of your service.
- **Jira Data Center** (optional, ask): searching is allowed; creating issues and comments asks you first. Without it the agent cannot see existing tickets, so its drafts may duplicate one, and it cannot learn from resolved incidents; it says so in its report.
- **GitLab** (optional, read): lets it read your service's configuration (Puppet, Helm charts, application code) when a finding points at a deployment or a limit.
- **A service memory**, written by you. This is the one thing the agent cannot do for itself, and it matters more than anything else here. See below.

## Setting it up

1. **Create the agent.** Nothing is asked of you; the connections are the only choice.
2. **Write the service memory.** The agent finds a service by its memory: the name after `service-` *is* the service's name. The template includes `service-example`, an example of what to put in one: the service's components and hosts, the Grafana data sources (name and uid), the Prometheus selectors and useful metrics, the OpenSearch index and fields, dashboards, the Jira project key, normal behaviour and known issues. Make a new memory called `service-<your service>` (a short lowercase name, such as `service-mirror`) with the same structure, filled with facts about your service, and **lock it** so the agent can read it but never change it. Delete `service-example` when you are done.
3. **Try it.** Ask the agent in a conversation for a situation report on your service. Fix what it could not find by improving the service memory. Ask it to *draft* tickets until you trust them.
4. **Turn on the schedule.** The template creates a disabled **Nightly check** schedule (06:00 every day, in the workspace's time zone). Enable it when the memory is ready.

## What it does on its own

Only the **Nightly check** schedule, once you enable it: a situation report over the last 12 hours for every service that has a `service-<name>` memory (the example is skipped), with tickets filed or updated as described above.

## Adapting it

- **Services**: add one locked `service-<name>` memory per service and the nightly check covers it, with its own `nightmanager-<name>` notes. For separate schedules, times or ticket rules per service, edit the schedule's prompt or add another schedule that names the service. One agent per service is also fine.
- **Ticket rules**: the confidence threshold and the limit of three new tickets per run come from the `jira-ticketing` skill and can be changed in the schedule's prompt ("file only high-confidence tickets", "draft only").
- **Memory care**: the agent's own memory, `nightmanager-<service>`, is kept under 8,000 characters and tidied automatically (after any save, and once a month). Read it from time to time: remove anything wrong, and write down corrections the agent could not know. Anything it records that contradicts your service memory is shown as a discrepancy in its next report.
- **Instructions and skills** can be edited like any agent's. Updates from newer template versions are offered, never applied automatically; memories are never offered again.
