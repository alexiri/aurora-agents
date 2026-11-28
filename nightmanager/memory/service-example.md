---
key: service-example
updated: 2026-11-25
locked: true
---
EXAMPLE: this memory only shows the format of an operator memory. It describes no real service. Your agent ignores it. Copy its shape to a new memory named `service-<your service>` (a short lowercase name, such as `service-mirror`), replace every line with facts about your service, lock it, and delete this one.

# Service: Example mirror

Public package mirror at mirror.example.org. Owned by the Platform Services team; on-call contact: platform-oncall@example.org.

Runs on 6 virtual machines (hostgroup `mirror/web`, Apache httpd 2.4) behind a load balancer. Files are served from one shared network filesystem mounted at `/mnt/data` on every host. `mirror.example.org` is a DNS alias of the load balancer.

## Dependencies
- Shared filesystem (`/mnt/data`): if it is slow, every Apache worker blocks and load rises with idle CPU.
- Load balancer: health-checks `/healthz` on each host; a failing host leaves the pool after 3 failed checks.
- Nightly sync jobs (scheduler, 02:00 UTC) pull upstream repositories onto the shared filesystem.

## Metrics (Prometheus)
Grafana data source: `Metrics Prometheus`, uid `<prometheus-datasource-uid>`.

Select hosts with `hostgroup=~"mirror/web.*"`.

Useful metrics: `apache_up`, `apache_workers{state="busy"|"idle"}`, `apache_accesses_total`, `apache_sent_kilobytes_total`, `apache_uptime_seconds_total` (use `resets()` to spot restarts), `node_load1`, `node_network_transmit_bytes_total`, `node_filesystem_avail_bytes{mountpoint="/mnt/data"}`.

Apache worker cap: `MaxRequestWorkers = 300` per host. Busy workers never exceed it, so a sample of 300 means the host is full.

## Logs (OpenSearch)
Grafana data source: `Logs web`, uid `<opensearch-datasource-uid>`, index pattern `mirror_logs_web*`.

- Timestamp field: `metadata.timestamp`.
- Fields: `data.clientip` (IP), `data.path`, `data.apache_response` (number), `data.return_data_size` (response bytes; NOTE: `data.bytes` does not exist), `data.response_time` (microseconds), `data.agent` (user agent), `data.logsource` (serving host).
- Cardinality aggregations on `data.clientip` work.
- Logged bytes can exceed what crossed the network: aborted downloads are logged in full.

## Dashboards (Grafana folder "Mirror")
- Apache `<dashboard-uid>`: workers, traffic, per-code and per-client log panels.
- Sync jobs `<dashboard-uid>`: status of the nightly sync jobs; "Failing jobs" alerts land here.

## Jira
Project `MIR` ("Mirror") on the company Jira.

## Repositories (where the configuration lives)
- `platform/mirror-puppet`: Puppet code and Hiera data for the web hosts. Apache settings (workers, timeouts, rate limits, access rules) are in `hieradata/hostgroup/mirror/web.yaml`.
- `platform/mirror-sync-jobs`: Nomad job definitions of the nightly sync, one file per upstream (the "Failing jobs" alerts).
- Name these in tickets, so a fix can be made from the ticket alone.

## Normal behaviour (weekday, observed 2026-11-10..14; a single week, so treat it as guidance)
- All hosts `apache_up=1`; no restarts; no 5xx.
- Requests: 70,000–140,000 per 15 minutes fleet-wide.
- Busy workers: 50–135 fleet-wide (typical 60–100); a host above 250 for more than 5 minutes is not normal.
- Egress: 15–45 MB/s fleet-wide.
- 404s: about 7% of requests, mostly clients asking for removed repositories (known issue below).
- The nightly sync jobs at 02:00 UTC raise filesystem metadata load for about 20 minutes.

## Alerts (for the watch-for-recurrence skill)
- Grafana folder for the agent's tripwire rules: `Mirror agent rules`, uid `<folder-uid>`.
- Contact point that calls the agent's webhook: `aurora-nightmanager`.
- Prometheus data source uid for tripwire expressions: `<prometheus-datasource-uid>` (the `Metrics Prometheus` source above).
- OpenSearch data source uid for log tripwires: `<opensearch-datasource-uid>` (the `Logs web` source above).
- Tripwires the operator allows for this service: 5.
- Create nothing for this service if any of these is missing.

## Known issues
- Two internal clients poll `repomd.xml` of removed repositories around the clock: a large, constant share of all 404s. Harmless; do not report it unless it changes.
- Requests that begin with a double slash (`//repo/...`) download the same file twice. Cosmetic.
- Internet scanners probe for `.env` and similar files. Low volume; ignore.
- The "Failing jobs" alert fires for the same 12 sync jobs about twice a day because their upstream is slow. It resolves by itself within 15 minutes.
