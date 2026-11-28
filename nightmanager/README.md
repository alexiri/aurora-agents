# NightManager

## What it does

NightManager is a Site Reliability Engineering agent that looks after one service at a time while you are away:

- **Investigates** the service over a time window (the last 12 hours by default). It queries Prometheus metrics and OpenSearch logs through Grafana, compares them with the same window a day and a week earlier, and writes a **situation report**: status (OK, DEGRADED, DOWN or UNKNOWN), numbered observations with the exact queries and returned values, a timeline, hypotheses with a confidence and the evidence for and against, suggested checks, and the data it could not get.
- **Files Jira tickets** for hypotheses that are specific enough to act on and confident enough (medium by default). It first looks for an existing ticket and updates that instead of filing a duplicate. Every ticket it files carries the label `nightmanager`.
- **Keeps notes** about each service: baselines of normal behaviour, confirmed failure modes, what became of its tickets. It keeps those notes small by merging repeats and dropping what has stopped mattering.

It reads; it does not fix. It cannot change dashboards, silences or any other Grafana or monitoring state, and it makes no change to your services — the one exception is **tripwires** (below), its own alert rules, every one of which waits for your approval. It never messages anyone on its own. Creating or commenting on a Jira ticket waits for your approval, because the Jira connection is set to ask. **Jira is optional**: without it the agent only reports, and puts ticket drafts in the report.

## Who it's for

Teams who run a service with Prometheus metrics and OpenSearch logs in Grafana and track their work in Jira, and who want a first look at overnight or weekend trouble before an engineer starts: what happened, how sure the agent is, and what a ticket should say.

## Working with The Fixer

NightManager has a teammate: **The Fixer** (another template in this repository). NightManager files tickets with the label `nightmanager`; a triager adds `fixer-ready` to the ones worth a try; The Fixer opens a draft merge request and labels the ticket `fixer-mr`; a person reviews it; The Fixer says on the ticket when it's merged (or closed); and NightManager then runs the ticket's verification query from the merge time and says whether the fix worked. Neither agent changes a ticket's status, and only a person adds `fixer-ready`. Use the same service name in both: NightManager reads `service-<name>`, The Fixer reads `repos-<name>`. Name the repositories where the configuration lives in your `service-<name>` memory so tickets can say where a change belongs. NightManager works fine without The Fixer.

## What it needs

- **Grafana** (custom): the agent runs its queries as the account you give it and reads dashboards, alert state and contact points; it may also set **tripwire** alert rules, and every create, change or delete of one waits for your approval. Give it a service account that can see the data sources of your service; for tripwires, grafana 1.6.0 or later and a token that can edit **one folder** (see Tripwires).
- **Jira Data Center** (optional, ask): searching is allowed; creating issues and comments asks you first. Without it the agent cannot see existing tickets, so its drafts may duplicate one, and it cannot learn from resolved incidents; it says so in its report.
- **GitLab** (optional, read): lets it read your service's configuration (Puppet, Helm charts, application code) when a finding points at a deployment or a limit.
- **A service memory**, written by you. This is the one thing the agent cannot do for itself, and it matters more than anything else here. See below.

## Setting it up

1. **Create the agent.** Nothing is asked of you; the connections are the only choice.
2. **Write the service memory.** The agent finds a service by its memory: the name after `service-` *is* the service's name. The template includes `service-example`, an example of what to put in one: the service's components and hosts, the Grafana data sources (name and uid), the Prometheus selectors and useful metrics, the OpenSearch index and fields, dashboards, the Jira project key, normal behaviour and known issues. Make a new memory called `service-<your service>` (a short lowercase name, such as `service-mirror`) with the same structure, filled with facts about your service, and **lock it** so the agent can read it but never change it. Delete `service-example` when you are done.
3. **Try it.** Ask the agent in a conversation for a situation report on your service. Fix what it could not find by improving the service memory. Ask it to *draft* tickets until you trust them.
4. **Turn on the schedule.** The template creates a disabled **Nightly check** schedule (06:00 every day, in the workspace's time zone). Enable it when the memory is ready.

## What it does on its own

The **Nightly check** schedule, once you enable it: a situation report over the last 12 hours for every service that has a `service-<name>` memory (the example is skipped), with tickets filed or updated as described above. And a **wake-up**: when a tripwire fires, Grafana calls the agent's **Grafana alert** webhook and the agent investigates in the moment (below).

## Tripwires

A tripwire is a Grafana alert rule the agent sets for the signature of a problem it has already evidenced — a ticket it filed or a failure mode it recorded — so that Grafana wakes it when the problem comes back. The agent uses them sparingly: at most 5 per service and 10 in total, every one with an expiry of at most 30 days, and one tripwire wakes it at most 3 times in 24 hours (after that it only counts and asks for a person on the ticket). A wake-up investigates briefly, comments on the ticket and changes nothing on any service.

What a tripwire needs, in order:

1. **Grafana 1.6.0 or later** on the connected server (older versions have no alert-rule tools; the agent reports the gap and sets nothing).
2. **A folder named for the agent**, with the service account's token allowed to edit that one folder and no other (in Grafana: create the folder, open it, Permissions, add the service account with Edit). One folder per agent: every NightManager agent carries the same `managed_by` label, so two agents sharing a folder would delete each other's rules in the expiry clean-up and count each other's rules toward the caps.
3. **The folder, the contact point's name and the Prometheus data source uid** written in the operator memory's "Alerts" section — the agent creates nothing when any of these is missing.
4. **A contact point** in Grafana that calls the agent's webhook. This is the only thing you create in Grafana for the wake-ups.
   - In Aurora, the template creates the **Grafana alert** webhook paused. Copy its address and the secret it shows once, and leave it paused for now.
   - In Grafana (version 11 or later), open *Alerts & IRM*, *Alerting*, *Notification configuration*, the *Contact points* tab, and choose **+ Add contact point**. Fill the form in this order:
     - **Name**: exactly the contact point name you wrote in the operator memory (`aurora-nightmanager` in the example). The agent's rules refer to the contact point by this name.
     - **Integration**: **Webhook**.
     - **URL**: the Aurora address. Never put the secret in the URL.
     - **HTTP Method**: **POST**.
     - **Authentication Header Scheme**: `Bearer`, Grafana's default.
     - **Authentication Header Credentials**: the secret. Don't use Basic Authentication, because Grafana takes either that or the header, not both.
     - **Max Alerts**: **10**. One notification then carries at most 10 alerts; any beyond that are left out, and the payload's `truncatedAlerts` says how many. The agent reads that number and looks the rest up itself. `0` means no limit: don't use it, because a flood of alerts would arrive as one very large message.
     - **Disable resolved message**: **on**. A "resolved" notice would wake the agent for nothing.
     - Leave everything else at its default: the title and message templates (the agent reads Grafana's default JSON payload) and the TLS settings. Don't use the HMAC signature settings, because Aurora can't verify Grafana's signature.
   - Choose **Test**, then **Save contact point**.
   - **You don't need a notification policy.** Every rule the agent creates names this contact point in its own notification settings (Grafana's simplified routing: in the rule form, *Configure notifications*, **Default**). Grafana sends that rule's alerts straight to the contact point, and notification policies, their matchers and their mute timings don't apply to it. The agent never creates or changes contact points, notification policies, mute timings or silences.
   - Timing comes from Grafana's defaults for such rules, and the agent can't change it: group wait 30 seconds, group interval 5 minutes, repeat interval 4 hours. So a tripwire that stays firing reaches the agent about every 4 hours, within the 3-a-day cap above.
   - The Grafana app's setup text in Aurora has the same steps, with a checklist and a list of what to look at when nothing arrives.
5. **Then enable the Grafana alert webhook** in Aurora, and use its test button: the webhook page shows recent deliveries, so you can confirm the whole path before any tripwire exists.

Every tripwire is one query Grafana evaluates on its interval, and every wake-up is a full agent turn — that is what the caps and the expiry are for. You can see the agent's rules in Grafana's alert list filtered by the folder, edit or delete them like any rule, and pause the webhook or delete the contact point to stop the wake-ups. Rules do not expire by themselves: the agent deletes its own expired rules at the start of its runs (each delete waits for your approval — approve it, or pause the webhook if you want the wake-ups stopped first).

## Adapting it

- **Services**: add one locked `service-<name>` memory per service and the nightly check covers it, with its own `nightmanager-<name>` notes. For separate schedules, times or ticket rules per service, edit the schedule's prompt or add another schedule that names the service. One agent per service is also fine.
- **Ticket rules**: the confidence threshold and the limit of three new tickets per run come from the `jira-ticketing` skill and can be changed in the schedule's prompt ("file only high-confidence tickets", "draft only").
- **Memory care**: the agent's own memory, `nightmanager-<service>`, is kept under 8,000 characters and tidied automatically (after any save, and once a month). Read it from time to time: remove anything wrong, and write down corrections the agent could not know. Anything it records that contradicts your service memory is shown as a discrepancy in its next report.
- **Instructions and skills** can be edited like any agent's. Updates from newer template versions are offered, never applied automatically; memories are never offered again.
