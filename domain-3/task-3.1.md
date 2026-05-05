# Task 3.1: Configure CLAUDE.md files with appropriate hierarchy, scoping, and modular organization

### Knowledge of:

- **The CLAUDE.md configuration hierarchy: user-level (`~/.claude/CLAUDE.md`), project-level (`.claude/CLAUDE.md` or root `CLAUDE.md`), and directory-level (subdirectory `CLAUDE.md` files)**

  Three levels, from broadest to most specific:
  1. **User-level** (`~/.claude/CLAUDE.md`) — personal preferences, applies to all projects
  2. **Project-level** (`.claude/CLAUDE.md` or root `CLAUDE.md`) — team standards, shared via git
  3. **Directory-level** (subdirectory `CLAUDE.md`) — area-specific conventions

- **That user-level settings apply only to that user — instructions in `~/.claude/CLAUDE.md` are not shared with teammates via version control**

  User-level config is for personal preferences (editor style, preferred patterns). Never put team standards here.

- **The `@import` syntax for referencing external files to keep CLAUDE.md modular (e.g., importing specific standards files relevant to each package)**

  ```markdown
  @import ./standards/api-conventions.md
  @import ./standards/testing.md
  ```
  Keeps the main CLAUDE.md focused; detailed conventions live in separate files.

- **`.claude/rules/` directory for organizing topic-specific rule files as an alternative to a monolithic CLAUDE.md**

  Instead of one huge CLAUDE.md, split into:
  - `.claude/rules/testing.md`
  - `.claude/rules/api-conventions.md`
  - `.claude/rules/deployment.md`

### Skills in:

- **Diagnosing configuration hierarchy issues (e.g., a new team member not receiving instructions because they're in user-level rather than project-level configuration)**

  If a teammate doesn't see instructions, check if they're in `~/.claude/CLAUDE.md` (user-only) instead of project-level config.

- **Using `@import` to selectively include relevant standards files in each package's CLAUDE.md based on maintainer domain knowledge**

  Each package imports only the standards relevant to it. The API package imports API conventions; the UI package imports design guidelines.

  Example monorepo `packages/api/CLAUDE.md`:

  ```markdown
  # API Package

  @import ../../standards/api-conventions.md
  @import ../../standards/testing.md
  @import ../../standards/error-handling.md

  ## Package-specific notes
  - Uses Express with Zod validation middleware
  - All routes must have OpenAPI annotations
  ```

  Example monorepo `packages/ui/CLAUDE.md`:

  ```markdown
  # UI Package

  @import ../../standards/testing.md
  @import ../../standards/accessibility.md

  ## Package-specific notes
  - React 19 with server components
  - All interactive elements need aria-labels
  ```

- **Splitting large CLAUDE.md files into focused topic-specific files in `.claude/rules/` (e.g., `testing.md`, `api-conventions.md`, `deployment.md`)**

  Break up monolithic files for maintainability. Each rule file is self-contained.

  Example `.claude/rules/` directory structure:

  ```
  .claude/rules/
  ├── testing.md          # Test patterns, fixtures, coverage thresholds
  ├── api-conventions.md  # REST naming, pagination, error format
  ├── deployment.md       # CI gates, env vars, rollback procedures
  └── security.md         # Auth requirements, input validation, secrets
  ```

  Example `.claude/rules/testing.md`:

  ```markdown
  # Testing Standards

  - Use vitest for unit tests, playwright for e2e
  - Test files live next to source: `foo.ts` -> `foo.test.ts`
  - Minimum 80% branch coverage for new code
  - Use `createTestFixture()` from `test/helpers.ts` for DB state
  - Never mock the database in integration tests; use test containers
  ```

  All files in `.claude/rules/` are automatically loaded — no `@import` needed.

- **Using the `/memory` command to browse and edit auto-memory files, and diagnose inconsistent behavior across sessions**

  If Claude isn't following conventions, use `/memory` to browse what Claude has saved (auto-memory notes from corrections and preferences). Everything is plain markdown you can read, edit, or delete. For CLAUDE.md specifically, verify the file exists at the expected path and level in the hierarchy.
