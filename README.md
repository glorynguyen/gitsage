# GitSage - AI-Powered Code Review MCP Server

A Model Context Protocol (MCP) server that provides AI-powered code review tools for Claude Desktop and Claude Code. Get intelligent code reviews, commit message generation, and code analysis directly in your AI assistant.

## Features

- **Code Review Tools**: Review staged changes, specific commits, commit ranges, or branch comparisons
- **Commit Message Generation**: Generate conventional commit messages from diffs
- **Code Analysis**: Explain code, suggest refactoring, generate tests, fix issues, generate documentation
- **Agent Skills**: Apply specialized review skills from GitHub repositories (e.g., security, performance)
- **Git Integration**: Full git repository access with status, commits, and branches
- **Diff Filtering**: Automatically filters out noise (lock files, generated code, minified files)

## Installation

### Prerequisites

- Node.js 18+
- npm
- Git

### Install from npm (recommended)

```bash
npm install -g gitsage
```

Or run without installing:

```bash
npx gitsage
```

### Install from source

```bash
git clone https://github.com/glorynguyen/gitsage.git
cd gitsage
npm install
npm run build
npm install -g .  # optional: install globally from source
```

## Configuration

### Claude Desktop

Add the server to your Claude Desktop configuration file:

**macOS**: `~/Library/Application Support/Claude/claude_desktop_config.json`
**Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

Using npx (recommended, no install required):

```json
{
  "mcpServers": {
    "gitsage": {
      "command": "npx",
      "args": ["gitsage"]
    }
  }
}
```

Or if installed globally via npm:

```json
{
  "mcpServers": {
    "gitsage": {
      "command": "gitsage"
    }
  }
}
```

Or from source with environment variables:

```json
{
  "mcpServers": {
    "gitsage": {
      "command": "node",
      "args": ["/path/to/gitsage/dist/index.js"],
      "env": {
        "GITHUB_TOKEN": "your-github-token-optional",
        "CODE_REVIEW_FRAMEWORKS": "React,TypeScript,Node.js",
        "CODE_REVIEW_WORKING_DIR": "/path/to/default/repo"
      }
    }
  }
}
```

### Claude Code

Add to your `~/.claude.json` (project) or `~/.claude/settings.json` (global):

```json
{
  "mcpServers": {
    "gitsage": {
      "command": "npx",
      "args": ["gitsage"]
    }
  }
}
```

Or if installed globally:

```json
{
  "mcpServers": {
    "gitsage": {
      "command": "gitsage"
    }
  }
}
```

Or from source with environment variables:

```json
{
  "mcpServers": {
    "gitsage": {
      "command": "node",
      "args": ["/path/to/gitsage/dist/index.js"],
      "env": {
        "GITHUB_TOKEN": "your-github-token-optional",
        "CODE_REVIEW_FRAMEWORKS": "React,TypeScript,Node.js",
        "CODE_REVIEW_WORKING_DIR": "/path/to/default/repo"
      }
    }
  }
}
```

Restart Claude Desktop or Claude Code after updating the config.

### Environment Variables

| Variable | Description |
|----------|-------------|
| `GITHUB_TOKEN` | GitHub token for skill fetching (avoids rate limits) |
| `CODE_REVIEW_FRAMEWORKS` | Comma-separated list of frameworks for review context |
| `CODE_REVIEW_WORKING_DIR` | Default repository path |

### Config File

Create `~/.config/gitsage/config.json`:

```json
{
  "frameworks": ["React", "TypeScript", "Node.js"],
  "skillRepositories": ["vercel-labs/agent-skills"],
  "diffFilter": {
    "ignorePaths": ["node_modules/**", "dist/**"],
    "ignorePatterns": ["*.min.js", "*.map"],
    "maxFileLines": 500,
    "ignoreFormattingOnly": false
  }
}
```

## Available Tools

### Code Review

| Tool | Description |
|------|-------------|
| `review_staged_changes` | Review staged git changes |
| `review_commit` | Review a specific commit by hash |
| `review_commit_range` | Review changes between two commits |
| `review_branches` | Compare two branches |

### Code Analysis

| Tool | Description |
|------|-------------|
| `generate_commit_message` | Generate a conventional commit message |
| `explain_code` | Get a detailed explanation of code |
| `suggest_refactoring` | Get improvement suggestions |
| `generate_tests` | Generate unit tests |
| `fix_code` | Fix issues in code |
| `generate_documentation` | Generate JSDoc/TSDoc/docstrings |

### Skills Management

| Tool | Description |
|------|-------------|
| `list_skills` | List available agent skills |
| `select_skills` | Select skills for reviews |
| `clear_skills` | Clear selected skills |

### Git Information

| Tool | Description |
|------|-------------|
| `get_git_status` | Get repository status |
| `list_commits` | List recent commits |
| `list_branches` | List branches |

## Usage Examples

### Review Staged Changes

```
Review my staged changes in /path/to/my/project
```

### Review with Skills

```
First list available skills, then select 'security-review' and 'performance-review', then review my staged changes
```

### Generate Commit Message

```
Generate a commit message for my staged changes
```

### Compare Branches

```
Review the changes between main and feature/my-branch
```

## Resources

The server exposes these resources that Claude can read:

- `config://settings` - Current server configuration
- `skills://selected` - Currently selected skills
- `skills://downloaded` - All cached skills

## Development

```bash
# Watch mode for development
npm run watch

# Development with tsx (no build required)
npm run dev

# Build for production
npm run build

# Start the server
npm start
```

## Architecture

```
gitsage/
├── src/
│   ├── index.ts       # MCP server entry point & tool handlers
│   ├── prompts.ts     # Prompt templates for AI interactions
│   ├── skills.ts      # Skills service (GitHub integration)
│   ├── git.ts         # Git operations (simple-git wrapper)
│   ├── diffFilter.ts  # Diff filtering logic
│   └── config.ts      # Configuration management
├── dist/              # Compiled output
├── package.json
└── tsconfig.json
```

## License

MIT
