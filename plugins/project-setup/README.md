# project-setup

An intelligent project bootstrapper for Claude Code. When invoked, this skill:

1. **Interviews you** about your project — what it does, your stack, what capabilities you need
2. **Researches** the best tools using context7 (live docs) and GitHub (community resources)
3. **Recommends** MCP servers, plugins, hooks, and skills tailored to your stack
4. **Scaffolds** the complete folder structure, CLAUDE.md, .mcp.json, and settings.json
5. **Offers visualization** of your architecture using the playground concept-map

## Requirements

- context7 MCP server (for live documentation lookup)
- github MCP plugin (for community resource discovery)

Both are available via the official Claude Code marketplace:

```
/plugin install context7@claude-plugins-official
/plugin install github@claude-plugins-official
```

## Usage

After installing this plugin, start a new project session and say:

> "Set up my project" or "Bootstrap this project" or "Initialize a new [type] project"

The skill will guide you through the rest.

## Visualization

After setup, the skill can generate:
- An **interactive concept map** of your architecture (requires `playground` plugin)
- **UI mockups** of frontend components (requires `frontend-design` plugin)

Install them from the official marketplace if you want visualization:

```
/plugin install playground@claude-plugins-official
/plugin install frontend-design@claude-plugins-official
```
