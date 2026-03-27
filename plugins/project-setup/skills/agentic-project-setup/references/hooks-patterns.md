# Hooks Patterns

Reference: load this file when recommending or writing hooks for a multi-agent orchestration project.

Hooks go in `.claude/settings.json` under the `hooks` key.

## Format

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "your-command-here"
          }
        ]
      }
    ]
  }
}
```

## Common Patterns by Stack

### Python — Ruff lint + format on every edit

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "ruff check --fix \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null; ruff format \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null; true"
          }
        ]
      }
    ]
  }
}
```

### JavaScript / TypeScript — ESLint + Prettier on every edit

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx eslint --fix \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null; npx prettier --write \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null; true"
          }
        ]
      }
    ]
  }
}
```

### Protect .env files from edits

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$CLAUDE_TOOL_INPUT_FILE_PATH\" | grep -q '\\.env'; then echo 'Blocked: do not edit .env files directly'; exit 2; fi"
          }
        ]
      }
    ]
  }
}
```

### Protect lock files from edits

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$CLAUDE_TOOL_INPUT_FILE_PATH\" | grep -qE '(package-lock\\.json|yarn\\.lock|pnpm-lock\\.yaml|poetry\\.lock|Cargo\\.lock)'; then echo 'Blocked: do not edit lock files directly'; exit 2; fi"
          }
        ]
      }
    ]
  }
}
```

### Agentic-specific: Validate prompt files aren't empty on write

Prevent accidentally writing an empty prompt file (which would silently break an agent):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "if echo \"$CLAUDE_TOOL_INPUT_FILE_PATH\" | grep -q 'prompts/'; then if [ ! -s \"$CLAUDE_TOOL_INPUT_FILE_PATH\" ]; then echo 'Warning: prompt file is empty — agent will have no system prompt'; exit 2; fi; fi; true"
          }
        ]
      }
    ]
  }
}
```
