# Plugin Catalog

Reference: load this file when recommending Claude Code plugins based on project signals.

## Signal → Plugin Mapping

| Project Signal | Plugin | What it adds |
|----------------|--------|-------------|
| Needs git commits managed | **commit-commands** | `/commit`, `/clean-gone`, `/commit-push-pr` commands |
| Building features iteratively | **feature-dev** | `/feature-dev` workflow: architect → explorer → reviewer agents |
| Wants automated hooks | **hookify** | `/hookify` configures hooks via conversation |
| Building another plugin/skill | **plugin-dev** | 7 skills covering hook-dev, MCP integration, skill creation |
| Needs code reviewed | **code-review** | `/code-review` command with review agent |
| Building MCP server | **mcp-server-dev** | Skills for building local/remote MCP servers |
| Frontend / React / Vue work | **frontend-design** | Skill for production-grade, distinctive UI generation |
| Wants interactive architecture visualization | **playground** | Concept maps, code maps, data explorers as HTML playgrounds |
| Building with Claude Agent SDK | **agent-sdk-dev** | SDK templates, `/new-sdk-app` command, verifier agents |
| Wants CLAUDE.md managed | **claude-md-management** | Skill to audit and improve CLAUDE.md files |
| Wants PR reviews | **pr-review-toolkit** | PR review workflow with structured feedback |

## Install Command

```bash
/plugin install {plugin-name}@claude-plugins-official
```

## settings.json enabledPlugins

After install, plugins can be scoped to a project in `.claude/settings.json`:

```json
{
  "enabledPlugins": {
    "commit-commands@claude-plugins-official": true,
    "feature-dev@claude-plugins-official": true
  }
}
```
