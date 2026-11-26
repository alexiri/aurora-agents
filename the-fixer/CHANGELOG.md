## 1.1.0

- Takes work from other agents: a peer task counts as a triager's own request when it comes from an agent listed under Trusted agents in the repository map and names a listed triager. Add a **Trusted agents** line to your `repos-<service>` memory (see `repos-example`); without it, no agent can hand it work. Limits and refusals are unchanged.

## 1.0.0

- First release. Takes tickets a triager has marked `fixer-ready`, makes the fix in the repositories listed in a repository map, and opens a draft merge request. Works with NightManager 1.3.0 or later.
