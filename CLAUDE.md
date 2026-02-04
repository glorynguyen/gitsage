# CLAUDE.md - GitSage Project Knowledge

This file provides context for Claude Code when working on this project.

## Project Overview

GitSage is a Model Context Protocol (MCP) server that provides AI-powered code review tools. It integrates with Claude Desktop and Claude Code to offer intelligent code analysis directly in AI assistants.

## Installation

```bash
# Install globally from npm
npm install -g @vinhnguyen/gitsage

# Or run without installing
npx @vinhnguyen/gitsage
```

## Tech Stack

- **Runtime**: Node.js 18+
- **Language**: TypeScript (ES2022, NodeNext modules)
- **Build**: tsc (TypeScript compiler)
- **Key Dependencies**:
  - `@modelcontextprotocol/sdk` - MCP server implementation
  - `simple-git` - Git operations
  - `@octokit/rest` - GitHub API for skills
  - `js-yaml` - YAML parsing for skill metadata
  - `minimatch` - Glob pattern matching for diff filtering

## Project Structure

```
src/
├── index.ts       # MCP server entry point, tool/resource/prompt handlers
├── git.ts         # GitService class wrapping simple-git
├── prompts.ts     # Prompt templates for code review, commit messages, etc.
├── skills.ts      # SkillsService for fetching skills from GitHub
├── diffFilter.ts  # Filters noise from git diffs (lock files, generated code)
└── config.ts      # Configuration management (file + env vars)
```

## Key Concepts

### MCP Tools (16 total)
The server exposes these tool categories:
- **Code Review**: `review_staged_changes`, `review_commit`, `review_commit_range`, `review_branches`
- **Code Analysis**: `explain_code`, `suggest_refactoring`, `generate_tests`, `fix_code`, `generate_documentation`
- **Commit**: `generate_commit_message`
- **Skills**: `list_skills`, `select_skills`, `clear_skills`
- **Git Info**: `get_git_status`, `list_commits`, `list_branches`

### MCP Resources
- `config://settings` - Server configuration
- `skills://selected` - Currently selected skills
- `skills://downloaded` - Cached skills

### MCP Prompts
- `code_review` - Review diff with framework context
- `commit_message` - Generate commit message from diff

### Skills System
Skills are review guidelines fetched from GitHub repositories (default: `vercel-labs/agent-skills`). Each skill has:
- YAML frontmatter with name/description
- Markdown body with review guidelines
- Cached locally in `~/.cache/gitsage/`

### Diff Filtering
The `diffFilter.ts` module removes noise from diffs:
- Ignores: `node_modules/**`, lock files, `dist/**`, `build/**`, `.next/**`
- Ignores patterns: `*.min.js`, `*.map`, `*.generated.*`
- Warns on large files (>500 lines by default)

## Configuration Locations

- **Config file**: `~/.config/gitsage/config.json`
- **Cache dir**: `~/.cache/gitsage/`
- **Environment variables**:
  - `GITHUB_TOKEN` - For GitHub API (skills fetching)
  - `CODE_REVIEW_FRAMEWORKS` - Comma-separated frameworks
  - `CODE_REVIEW_WORKING_DIR` - Default repo path

## Development Commands

```bash
npm run build    # Compile TypeScript to dist/
npm run watch    # Watch mode compilation
npm run dev      # Run with tsx (no build needed)
npm start        # Run compiled server
```

## Code Patterns

### Tool Handler Pattern
All tools follow this pattern in `index.ts`:
```typescript
async function handleToolName(
  args: Record<string, unknown>
): Promise<{ content: Array<{ type: string; text: string }> }> {
  // 1. Extract and validate args
  // 2. Create GitService if needed
  // 3. Perform operation
  // 4. Return formatted response
}
```

### GitService Usage
```typescript
const git = new GitService(workingDir);
if (!(await git.isValidRepo())) {
  throw new Error(`Not a valid git repository: ${workingDir}`);
}
const { diff, files, stats } = await git.getStagedDiff();
```

### Prompt Building
Prompts are built using template functions in `prompts.ts`:
```typescript
const prompt = buildReviewPrompt(diff, { frameworks, skills });
```

## Important Notes

- The server uses stdio transport (`StdioServerTransport`)
- All console output goes to stderr (stdout is for MCP protocol)
- Skills are cached to avoid GitHub rate limits
- The `selectedSkills` array persists for the session but not across restarts
- Diff filtering is applied before sending to AI to reduce token usage

## Testing Locally

1. Build: `npm run build`
2. Test directly: `echo '{"jsonrpc":"2.0","method":"tools/list","id":1}' | node dist/index.js`
3. Or use with Claude Desktop by adding to config
4. Or test via npx: `echo '{"jsonrpc":"2.0","method":"tools/list","id":1}' | npx @vinhnguyen/gitsage`

## Common Tasks

### Adding a New Tool
1. Add tool definition to `ListToolsRequestSchema` handler in `index.ts`
2. Create `handleNewTool` function
3. Add case to `CallToolRequestSchema` switch statement

### Modifying Prompts
Edit the template functions in `prompts.ts`. Each returns a string with embedded diff/code.

### Adding Diff Filters
Edit `DEFAULT_IGNORE_PATHS` or `DEFAULT_IGNORE_PATTERNS` in `diffFilter.ts`.
