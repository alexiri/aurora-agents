## 1.4.2

- The tripwire setup in the README names every setting of Grafana's webhook contact point, in the order of Grafana's form. It names **Max Alerts** as 10 and says what that does, and says **Disable resolved message** is on. It says plainly that the contact point is the only thing to create: no notification policy is needed, because the agent's rules name the contact point themselves, and the agent never creates or changes policies.
- When a wake-up's `truncatedAlerts` is above 0, the agent lists the firing alerts itself and matches them by the same rule, instead of missing the ones the notification left out.

## 1.4.0

- Tripwires: for a problem it has evidenced that could come back, the agent can set a managed Grafana alert rule that calls its webhook and wakes it to investigate in the moment. At most 5 tripwires per service and 10 in total, every one with an expiry, and a tripwire wakes the agent at most 3 times in 24 hours. The template now carries a **Grafana alert** webhook (created paused; a person turns it on after Grafana's contact point exists) and the agent's Grafana connection asks for approval on the three alert-rule tools; its read tools are narrower than a plain `read` level — only the query, dashboard and alert-state tools the agent needs.

## 1.3.0

- Works with The Fixer: tickets say where a change belongs and carry a verification query that can be run as written; a ticket labelled `fixer-mr` is left alone while its fix is in review; after the change is deployed, NightManager runs the verification query and reports whether it worked. Only a person adds `fixer-ready`.

## 1.2.0

- New, square avatar (256×256 PNG).

## 1.1.0

- Adds a default avatar. Agents created from the template start with it; people can change it afterwards.
