# Task 3.3: Apply path-specific rules for conditional convention loading

## Knowledge of:

### ◇ `.claude/rules/` files with YAML frontmatter `paths` fields containing glob patterns for conditional rule activation

```yaml
---
paths: ["src/api/**/*"]
---
# API Conventions
- Use async/await with explicit error handling
- Follow REST naming conventions
```
This rule only loads when editing files matching `src/api/**/*`.

### ◇ How path-scoped rules load only when editing matching files, reducing irrelevant context and token usage

No wasted tokens on API conventions when editing React components. Rules load only when relevant.

### ◇ The advantage of glob-pattern rules over directory-level CLAUDE.md files for conventions that span multiple directories (e.g., test files spread throughout a codebase)

Test files live everywhere (`src/auth/auth.test.tsx`, `src/api/orders.test.tsx`). A directory-level CLAUDE.md can't cover them all. A glob pattern `**/*.test.*` covers all test files regardless of location.

This is the answer to exam Question 6.

## Skills in:

### ◆ Creating `.claude/rules/` files with YAML frontmatter path scoping (e.g., `paths: ["terraform/**/*"]`) so rules load only when editing matching files

```yaml
---
paths: ["terraform/**/*"]
---
# Terraform Conventions
- Use modules for reusable infrastructure
- Pin provider versions
```

### ◆ Using glob patterns in path-specific rules to apply conventions to files by type regardless of directory location (e.g., `**/*.test.tsx` for all test files)

Type-based rules that work everywhere in the codebase, not just one directory.

### ◆ Choosing path-specific rules over subdirectory CLAUDE.md files when conventions must apply to files spread across the codebase

If your convention applies to a file *type* rather than a file *location*, use path-specific rules.
