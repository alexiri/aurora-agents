# Aurora agents

Ready-made agents for [Aurora](https://github.com/alexiri/aurora), kept as **template sources**: one directory per template, packaged and published by CI. The format is described in Aurora's `docs/25-agent-templates.md`.

## Templates

| Template | What it does |
|----------|--------------|
| [nightmanager](nightmanager/) | Watches one service at a time through Grafana (Prometheus and OpenSearch), writes evidence-based situation reports, files Jira tickets for actionable findings, and keeps small, tidy notes about each service. |

## Using the templates

An instance admin adds this repository as a template source in Aurora (`https://github.com/alexiri/aurora-agents`), reviews each version, and approves it. People then create the agent from the workspace's template gallery. Each template's own README says what it needs and how to set it up.

## Working on a template

```sh
aurora template check .            # the whole repository: every problem, with a path
aurora template pack nightmanager  # the package CI would publish, and its content hash
```

To publish a new version, raise `version` in `<template>/template.yaml` and merge: CI packs the template and creates the release `<id>-v<version>`. A published version is never replaced; changing a template means raising its version. Add release notes in `<template>/CHANGELOG.md`.

## Layout

```
aurora-templates.yaml             the repository manifest (where packages are published)
nightmanager/                     one directory per template, named by its id
  README.md  template.yaml  agent.yaml  instructions.md
  memory/  skills/  automations/
.github/workflows/aurora-templates.yml   check on every push, publish on merge to main
.github/aurora-publish.sh                the publish job's logic
```

## License

GPL-3.0, see [LICENSE](LICENSE).
