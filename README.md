# MH-Agents

Mapleton Hill's internal [Claude Code plugin](https://docs.claude.com/en/docs/claude-code/plugins)
marketplace. Private — installation is gated by GitHub access to this repo.

> **This local copy is specialized for the AVH Contentful migration.** The
> `pm-agent` plugin here is scoped down to Figma-driven ticket creation for
> that engagement; the general-purpose skills it started from (feature spec,
> backlog prioritization, SOW docs, status updates, actuals reporting) were
> moved to [`_archived-pm-agent-skills/`](./_archived-pm-agent-skills) rather
> than deleted, in case another engagement needs the fuller pipeline.

## Plugins

| Plugin | Description |
|---|---|
| [`pm-agent`](./pm_agent) | AVH Contentful migration toolkit — an Azure DevOps work-item agent that turns Figma designs into fully-specified tickets, plus drafting/triage/estimation skills. |

## Install

You need read access to this repo and working git auth for it (`gh auth login`,
SSH, or a credential helper — if `git clone` of this repo works, so will this).

```
/plugin marketplace add mapletonhillmedia/MH-Agents
/plugin install pm-agent@mh-agents
```

Then follow the plugin's own setup notes (e.g. `pm-agent` needs an
`AZURE_DEVOPS_PAT` env var — see [pm_agent/README.md](./pm_agent/README.md)).

### Enable for a whole project automatically

Commit this to a project's `.claude/settings.json` so teammates get the plugin
without running the commands above:

```json
{
  "extraKnownMarketplaces": {
    "mh-agents": {
      "source": { "source": "github", "repo": "mapletonhillmedia/MH-Agents" }
    }
  },
  "enabledPlugins": ["pm-agent@mh-agents"]
}
```

## Adding a plugin

1. Create a top-level folder for it (e.g. `my_plugin/`) with a
   `.claude-plugin/plugin.json` manifest.
2. Add `commands/`, `skills/`, `agents/`, and/or `.mcp.json` as needed.
3. Register it in [`.claude-plugin/marketplace.json`](./.claude-plugin/marketplace.json).
4. Never commit secrets — read tokens from environment variables in `.mcp.json`.
