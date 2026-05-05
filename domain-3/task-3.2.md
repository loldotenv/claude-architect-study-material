# Task 3.2: Create and configure custom slash commands and skills

## Knowledge of:

### ◇ Project-scoped commands in `.claude/commands/` (shared via version control) vs user-scoped commands in `~/.claude/commands/` (personal)

| Scope | Location | Shared? | Use for |
|---|---|---|---|
| Project | `.claude/commands/` | Yes (git) | Team workflows (`/review`, `/deploy`) |
| User | `~/.claude/commands/` | No | Personal shortcuts |

Exam Question 4: team-wide `/review` command -> `.claude/commands/` (answer A).

### ◇ Skills in `.claude/skills/` with `SKILL.md` files that support frontmatter configuration including `context: fork`, `allowed-tools`, and `argument-hint`

Skills are more advanced than commands. They support frontmatter for:
- `context: fork` — run in isolated subagent
- `allowed-tools` — restrict which tools the skill can use
- `argument-hint` — prompt for parameters when invoked without arguments

### ◇ The `context: fork` frontmatter option for running skills in an isolated subagent context, preventing skill outputs from polluting the main conversation

Use `fork` for skills that produce verbose output (codebase analysis, brainstorming). The output stays in the subagent; only the final result returns to the main session.

### ◇ Personal skill customization: creating personal variants in `~/.claude/skills/` with different names to avoid affecting teammates

Want to tweak a team skill? Create your own version in `~/.claude/skills/` with a different name. Teammates keep the original.

## Skills in:

### ◆ Creating project-scoped slash commands in `.claude/commands/` for team-wide availability via version control

Put it in `.claude/commands/review.md` and every teammate gets `/review` when they pull.

Example `.claude/commands/review.md`:

```markdown
Review the current git diff for:
1. Security vulnerabilities (injection, auth bypass)
2. Performance regressions (N+1 queries, missing indexes)
3. Missing error handling

Output a markdown table: | File | Line | Severity | Issue |
If no issues found, say "LGTM".
```

### ◆ Using `context: fork` to isolate skills that produce verbose output (e.g., codebase analysis) or exploratory context (e.g., brainstorming alternatives) from the main session

Fork prevents context pollution. The main session stays clean.

### ◆ Configuring `allowed-tools` in skill frontmatter to restrict tool access during skill execution (e.g., limiting to file write operations to prevent destructive actions)

Security boundary: limit what a skill can do.

Example `.claude/skills/SKILL.md` with full frontmatter:

```markdown
---
context: fork
allowed-tools: Read, Grep, Glob
argument-hint: "Provide the module path to analyze (e.g., src/auth)"
---

Analyze the specified module for architectural issues:
- Circular dependencies
- God classes (>300 lines)
- Missing interface abstractions

Write findings to ./analysis-report.md with recommendations.
```

The `context: fork` runs this in a subagent so verbose output stays isolated. `allowed-tools` restricts to read-only operations despite what the prompt says (the Write at the end would be blocked). `argument-hint` prompts the user if they invoke the skill without specifying a module.

### ◆ Using `argument-hint` frontmatter to prompt developers for required parameters when they invoke the skill without arguments

Makes skills user-friendly. `/deploy` without arguments prompts "Which environment? (staging/production)".

### ◆ Choosing between skills (on-demand invocation for task-specific workflows) and CLAUDE.md (always-loaded universal standards)

| Mechanism | When loaded | Use for |
|---|---|---|
| CLAUDE.md | Always | Universal standards, conventions |
| Skills | On-demand | Task-specific workflows |
