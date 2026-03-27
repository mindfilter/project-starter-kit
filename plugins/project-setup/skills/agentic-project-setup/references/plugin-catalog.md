# Plugin Catalog

Reference: load this file when recommending Claude Code plugins based on project signals.

## Signal → Plugin Mapping

| Project Signal | Plugin | What it adds |
|----------------|--------|-------------|
| Building with Claude Agent SDK | **agent-sdk-dev** | SDK templates, `/new-sdk-app` command, verifier agents for testing your agent implementation |
| Any multi-agent development workflow | **superpowers** | `dispatching-parallel-agents` and `subagent-driven-development` skills — enables using Claude Code itself as an orchestration tool while building the project |
| Building features iteratively (any team size) | **feature-dev** | `/feature-dev` workflow: architect → explorer → reviewer agents — mirrors the orchestration pattern you're building |
| Team needs git workflow | **commit-commands** | `/commit`, `/commit-push-pr` commands for clean git workflow on multi-contributor agent repos |
| Building another plugin/skill as part of project | **plugin-dev** | Skills covering hook-dev, MCP integration, skill creation |
| Needs code reviewed | **code-review** | `/code-review` command with review agent |
| Wants interactive architecture visualization | **playground** | Concept maps showing agent relationships, data flows, and orchestration structure |

## Install Command

```bash
/plugin install {plugin-name}@claude-plugins-official
```

## settings.json enabledPlugins

After install, plugins can be scoped to a project in `.claude/settings.json`:

```json
{
  "enabledPlugins": {
    "agent-sdk-dev@claude-plugins-official": true,
    "superpowers@claude-plugins-official": true,
    "feature-dev@claude-plugins-official": true
  }
}
```
