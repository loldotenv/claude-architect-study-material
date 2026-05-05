# Task 2.4: Integrate MCP servers into Claude Code and agent workflows

## Knowledge of:

### ◇ MCP server scoping: project-level (`.mcp.json`) for shared team tooling vs user-level (`~/.claude.json`) for personal/experimental servers

| Scope | File | Shared via git? | Use for |
|---|---|---|---|
| Project | `.mcp.json` | Yes | Team tools (Jira, DB, etc.) |
| User | `~/.claude.json` | No | Personal/experimental servers |

### ◇ Environment variable expansion in `.mcp.json` (e.g., `${GITHUB_TOKEN}`) for credential management without committing secrets

Store tokens in environment variables, reference them in `.mcp.json`. Secrets stay out of version control.

### ◇ That tools from all configured MCP servers are discovered at connection time and available simultaneously to the agent

When Claude Code starts, it connects to all configured MCP servers and makes all their tools available at once. No manual server selection needed.

### ◇ MCP resources as a mechanism for exposing content catalogs (e.g., issue summaries, documentation hierarchies, database schemas) to reduce exploratory tool calls

Resources let the agent browse what's available before making targeted tool calls, reducing wasteful exploration.

## Skills in:

### ◆ Configuring shared MCP servers in project-scoped `.mcp.json` with environment variable expansion for authentication tokens

```json
{
  "mcpServers": {
    "jira": {
      "command": "mcp-server-jira",
      "env": {
        "JIRA_TOKEN": "${JIRA_TOKEN}"
      }
    }
  }
}
```

### ◆ Configuring personal/experimental MCP servers in user-scoped `~/.claude.json`

Keep experimental servers out of the project config so they don't affect teammates.

**Example `~/.claude.json` with personal MCP servers:**

```json
{
  "mcpServers": {
    "my-notes": {
      "command": "mcp-server-obsidian",
      "args": ["--vault", "/Users/me/notes"],
      "env": {}
    },
    "local-llm-eval": {
      "command": "node",
      "args": ["/Users/me/dev/mcp-eval-server/index.js"],
      "env": {
        "EVAL_MODEL": "local"
      }
    },
    "github": {
      "command": "mcp-server-github",
      "env": {
        "GITHUB_TOKEN": "ghp_xxxxxxxxxxxx"
      }
    }
  }
}
```

Key points:
- This file is NOT committed to version control — it lives in your home directory
- Use it for personal productivity tools (note-taking, personal GitHub), experimental servers under development, or servers that require credentials you don't want to share
- Tools from these servers are available alongside project-scoped tools when working in any project

### ◆ Enhancing MCP tool descriptions to explain capabilities and outputs in detail, preventing the agent from preferring built-in tools (like Grep) over more capable MCP tools

Claude has strong priors toward built-in tools (Grep, Read, Bash) because it has seen them in training. An MCP tool with a vague description will lose the "selection competition" to a familiar built-in tool, even when the MCP tool is more capable.

**Before/After — a code search MCP server:**

```
BEFORE (loses to built-in Grep):
"search_codebase": "Searches code in the repository"

AFTER (wins over Grep when appropriate):
"search_codebase": "Semantic code search across the entire repository using
 embeddings. Unlike grep (which matches exact strings), this finds conceptually
 related code even with different naming. Use for: finding implementations of a
 concept, locating similar patterns, or discovering related functions.
 Input: { query: string (natural language description of what you're looking for),
          language?: string, max_results?: number (default 10) }
 Output: Array of { file_path, line_range, snippet, relevance_score }
 Prefer this over Grep when: searching by concept rather than exact string,
 looking for 'code that does X' rather than 'code containing string Y'."
```

The enhanced description explicitly compares itself to the built-in alternative, states when to prefer it, and documents its unique value (semantic vs. exact match).

### ◆ Choosing existing community MCP servers over custom implementations for standard integrations (e.g., Jira), reserving custom servers for team-specific workflows

The MCP ecosystem has mature community servers for common integrations. Building custom servers for standard tools wastes time and introduces maintenance burden.

**Examples of well-established community MCP servers:**

| Server | Package | Provides |
|---|---|---|
| **GitHub** | `@modelcontextprotocol/server-github` | Issues, PRs, repos, file contents, code search |
| **PostgreSQL** | `@modelcontextprotocol/server-postgres` | Schema inspection, read queries, table exploration |
| **Filesystem** | `@modelcontextprotocol/server-filesystem` | Sandboxed file read/write with configurable allowed directories |

**When to build custom instead:**
- Your team has a proprietary internal API (e.g., custom deployment system)
- You need business-logic-aware tools (e.g., "create release with our branching strategy")
- The community server lacks features specific to your workflow (e.g., Jira custom fields unique to your org)
- You need to enforce organization-specific validation or access controls at the tool level

### ◆ Exposing content catalogs as MCP resources to give agents visibility into available data without requiring exploratory tool calls

Resources = browsable catalogs. Tools = actions. Use resources to reduce the number of exploratory tool calls needed.
