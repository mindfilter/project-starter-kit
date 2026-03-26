# Hooks Patterns

Reference: load this file when recommending or writing hooks for a project.

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

### Rust — cargo fmt on every edit

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "cargo fmt 2>/dev/null; true"
          }
        ]
      }
    ]
  }
}
```

### Go — gofmt on every edit

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "gofmt -w \"$CLAUDE_TOOL_INPUT_FILE_PATH\" 2>/dev/null; true"
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
