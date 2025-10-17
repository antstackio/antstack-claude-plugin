# AntStack Claude Plugin

Structured workflow management for Claude Code with specialized agents, automated git workflows, and validation tools.

## Installation

### Method 1: From GitHub (Recommended)

```bash
# Add the plugin marketplace
/plugin marketplace add antstackio/antstack-claude-plugin

# Install the plugin
/plugin install antstack-claude-plugin@antstackio

# Enable the plugin
/plugin enable antstack-claude-plugin
```

### Method 2: Interactive Installation

```bash
# Browse and install interactively
/plugin
```

**Prerequisites:** Git, GitHub CLI (`gh`), `GITHUB_TOKEN` environment variable

## Quick Start

```bash
# 1. Start task
/start-task Add user authentication

# 2. Use agents
Use project-analyzer: Analyze authentication architecture
Use backend-developer: Implement JWT authentication API
Use frontend-developer: Create login form component

# 3. Validate
/validate

# 4. Complete
/complete-task
```

## Features

### Three Specialized Agents

| Agent                  | Model  | Use For                                             |
| ---------------------- | ------ | --------------------------------------------------- |
| **project-analyzer**   | Opus   | Architecture analysis, technical debt, code quality |
| **backend-developer**  | Sonnet | AWS Lambda, APIs, databases, authentication         |
| **frontend-developer** | Sonnet | React, Next.js, UI components, forms                |

### Workflow Commands

- `/start-task <description>` - Initialize task with git workflow
- `/validate` - Run typecheck, lint, tests, build
- `/complete-task` - Commit, push, create PR
- `/sync-main` - Sync with remote
- `/create-pr` - Create pull request

### MCP Integrations

GitHub • Context7 • AWS CDK • Playwright

### Automatic Hooks

Warns on temp file creation, blocks commits if temp files exist

## Configuration

### nvironment Variables

```bash
export GITHUB_TOKEN="ghp_your_token"
export AWS_REGION="us-east-1"  # optional
```

## Workflow Example

```bash
/start-task Implement notifications
Use project-analyzer: Analyze notification architecture
Use backend-developer: Create notification handlers
Use frontend-developer: Build notification UI
/validate
/complete-task
```

## Best Practices

1. **Always use Context7** for library documentation
2. **Start with project-analyzer** to understand architecture
3. **Run /validate** before completing tasks
4. **Let commands handle git** - focus on code

## Troubleshooting

**Commands not working?**

```bash
/plugin enable antstack-claude-plugin
```

**Validation failing?**
Fix in order: TypeScript → Linting → Tests → Build

**Git operations failing?**
Check `GITHUB_TOKEN` and GitHub CLI installation

## Contributing

Issues: https://github.com/antstackio/antstack-claude-plugin/issues

## License

MIT

## Version

1.0.0

---
