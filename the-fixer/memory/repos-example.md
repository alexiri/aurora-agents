---
key: repos-example
updated: 2026-11-24
locked: true
---
EXAMPLE: this memory only shows the format of the repository map. It describes no real service. Your agent ignores it. Copy its shape to a new memory named `repos-<your service>` (the same service name NightManager uses in `service-<your service>`, such as `repos-mirror`), replace every line with facts about your repositories, lock it, and delete this one.

# Repositories: Example mirror

Where the configuration of the Example mirror service (mirror.example.org) lives.

## Tickets
- **Ticket project:** `MIR`.
- **Triagers** (Jira user names; only these people's `fixer-ready` labels and comments steer the agent): `a.triager`, `b.triager` (the same names are used for their GitLab accounts unless a line here says otherwise). If this line is missing, the agent works on nothing.
- **After a merge:** Puppet applies the change to all web hosts within 30 minutes. Say so in the "Merged" comment, so the verification starts after that.

## Repositories

### platform/mirror-puppet (project id `<numeric id>`)
- **Covers:** the Puppet code and Hiera data for the web hosts (hostgroup `mirror/web`): Apache settings (workers, timeouts, rate limits, access rules), the load-balancer health check, monitoring agents. If a ticket is about how a host is configured, it's here.
- **Where:** `hieradata/hostgroup/mirror/web.yaml` for settings, `code/modules/mirror_web/` for templates and logic. Apache limits are the keys under `apache::`.
- **Branches:** default `main`. Merge requests target `main`. It's protected: direct pushes are refused.
- **Test:** `make lint` (about 1 minute: `puppet-lint`, `yamllint`) and `make spec` (about 4 minutes). Run both. `make spec` needs no network.
- **Conventions:** read `CONTRIBUTING.md`. Commit messages start with the ticket key. One setting per commit when you can. Changes to `hieradata/` don't need a version bump; changes to `code/` do (`metadata.json`).
- **Never touch:** `hieradata/secrets/`, anything under `hieradata/nodes/`, `.gitlab-ci.yml` and the files it includes (`ci/`).
- **Allowed here:** nothing beyond the defaults.

### platform/mirror-sync-jobs (project id `<numeric id>`)
- **Covers:** the Nomad job definitions for the nightly repository sync (the "Failing jobs" alerts). Schedules, retry settings, resource limits, one file per upstream.
- **Where:** `jobs/<upstream>.nomad.hcl`; shared defaults in `jobs/_defaults.hcl`.
- **Branches:** default `main`. Merge requests target `main`.
- **Test:** `make validate` (30 seconds: `nomad job validate` on every file, offline).
- **Conventions:** a retry or timeout change goes in `_defaults.hcl` only when it should apply to every job; otherwise change the single job file.
- **Never touch:** `jobs/credentials/`, `.gitlab-ci.yml`.

## Merge requests (all repositories)
- Labels: `automated`, `needs-review`. **Reviewers** (GitLab user names; their comments on a merge request steer the agent like the triagers' do): `x.reviewer`, `y.reviewer`. Leave reviewer assignment to GitLab's suggestions; don't pick people.
- Squash on merge. Delete the source branch on merge.
- Describe the effect on the hosts in the first line (what will change on the machines once it's deployed).

## Things to know
- Settings in `hieradata/` apply to all six web hosts at the next Puppet run (every 30 minutes). Say that in the merge request.
- `MaxRequestWorkers` is 300 and the hosts have 4 CPUs. Raising it is a decision for a person, not a fix: go ahead only when a triager's comment on the ticket names the new value; otherwise decline.
