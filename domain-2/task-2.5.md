# Task 2.5: Select and apply built-in tools (Read, Write, Edit, Bash, Grep, Glob) effectively

### Knowledge of:

- **Grep for content search (searching file contents for patterns like function names, error messages, or import statements)**

  Grep searches *inside* files for text patterns. It answers: "Which files contain this string/pattern?"

  **Decision heuristic:** You know *what* you're looking for (a function name, error message, import path) but not *where* it lives. Grep is the right choice.

- **Glob for file path pattern matching (finding files by name or extension patterns)**

  Glob matches file *paths* against naming patterns. It answers: "What files exist matching this structure?"

  **Decision heuristic:** You know the *type* of file (test files, config files, files in a specific directory) but not the exact content you need. Glob finds the candidates; then you Read them.

- **Read/Write for full file operations; Edit for targeted modifications using unique text matching**

  | Tool | Use when... | Fails when... |
  |---|---|---|
  | `Read` | You need to inspect file contents, understand context, or prepare for Edit | File doesn't exist |
  | `Write` | Creating new files, or modifying files where Edit cannot find unique anchors | — |
  | `Edit` | Making surgical changes to specific lines in an existing file | The `old_string` isn't unique in the file |

- **When Edit fails due to non-unique text matches, using Read + Write as a fallback for reliable file modifications**

  Edit requires a *unique* text match. Common failure cases:
  - Repeated patterns (e.g., multiple `import React from 'react'` lines)
  - Generic code (e.g., `return null;` appears in many functions)

  When Edit fails: Read the full file, make your changes to the content, Write the entire modified file back. This is always reliable but overwrites the whole file.

### Skills in:

- **Selecting Grep for searching code content across a codebase (e.g., finding all callers of a function, locating error messages)**

  **Tool selection decision flowchart:**

  ```
  "I need to find X in this codebase"
       │
       ├─ I know the exact string/pattern (function name, error message, import)
       │   → Grep
       │
       ├─ I know the file type/naming convention but not contents
       │   → Glob (then Read the results)
       │
       ├─ I know the exact file path
       │   → Read directly
       │
       ├─ I need to run a complex multi-step search (regex + filter + sort)
       │   → Bash (with grep, find, awk, etc.)
       │
       └─ I need to understand file structure / directory layout
           → Bash (with ls, tree, find)
  ```

  **Quick-reference table:**

  | Scenario | Tool | Example |
  |---|---|---|
  | Find all callers of `processPayment()` | Grep | `grep -r "processPayment" src/` |
  | Find all test files | Glob | `**/*.test.ts` |
  | Find all TypeScript files in the API layer | Glob | `src/api/**/*.ts` |
  | Find where an env var is used | Grep | `grep -r "DATABASE_URL"` |
  | Find the config file for ESLint | Glob | `**/.eslintrc*` or `**/eslint.config*` |
  | Find which file exports a specific type | Grep | `grep -r "export.*type UserProfile"` |
  | Count lines of code per directory | Bash | `find src -name "*.ts" \| xargs wc -l` |
  | Check if a dependency is used | Grep | `grep -r "from 'lodash"` |

- **Selecting Glob for finding files matching naming patterns (e.g., `**/*.test.tsx`)**

  Glob excels at structural queries about the codebase:
  - "What modules exist?" → `src/modules/*/index.ts`
  - "Are there migration files?" → `**/migrations/*.sql`
  - "What's the test coverage structure?" → `**/*.test.*` or `**/__tests__/**`

  **Glob vs Grep vs Bash decision:**
  - **Glob** if you're matching file *names/paths* with simple patterns
  - **Grep** if you're matching file *contents*
  - **Bash** (`find`) if you need complex file predicates (modified time, size, permissions, boolean combinations)

- **Using Read to load full file contents followed by Write when Edit cannot find unique anchor text**

  Prefer Edit when possible (it's safer — only changes what you target). Fall back to Read + Write when:
  - Edit reports "old_string not found" (typo or whitespace mismatch)
  - Edit reports "old_string not unique" (too common a pattern)
  - You need to make many changes throughout the file (multiple Edits vs. one Write)

- **Building codebase understanding incrementally: starting with Grep to find entry points, then using Read to follow imports and trace flows, rather than reading all files upfront**

  This is the most exam-relevant skill. The anti-pattern is reading every file in a directory "just in case." The correct approach is a targeted investigation:

  **Step-by-step example — "Understand how authentication works in this app":**

  1. **Glob for structure:** `**/auth*` or `**/middleware/*` — discover which files relate to auth
  2. **Grep for entry point:** `grep -r "authenticate\|isAuthenticated\|requireAuth" src/` — find where auth is invoked
  3. **Read the entry point:** Read the middleware file that applies auth checks (e.g., `src/middleware/auth.ts`)
  4. **Follow imports:** The file imports from `../services/token.ts` — Read that next
  5. **Grep for usage:** `grep -r "authMiddleware" src/routes/` — find which routes use it
  6. **Read selectively:** Only read the 2-3 route files that matter for your task

  **Why this matters:** Reading all files upfront wastes context window tokens on irrelevant code. In a large codebase (1000+ files), you physically cannot read everything. The incremental approach keeps your context focused and your tool calls efficient.

  **Exam connection (developer productivity scenario):** An agent asked to "fix the login bug" should NOT start by reading every file. It should Grep for the error message, Read the relevant handler, trace the auth flow, and fix the specific issue — mirroring how a senior developer would investigate.

- **Tracing function usage across wrapper modules by first identifying all exported names, then searching for each name across the codebase**

  This is a structured investigation pattern:

  1. **Read the module** to find its exports:
     ```
     Read src/utils/index.ts
     → exports: formatDate, parseCSV, slugify, debounce
     ```

  2. **Grep for each exported name** across the codebase:
     ```
     Grep "formatDate" → used in 12 files
     Grep "parseCSV" → used in 2 files
     Grep "slugify" → used in 0 files (dead code candidate!)
     Grep "debounce" → used in 8 files
     ```

  3. **Follow the usage chain** for the relevant function:
     ```
     Grep "parseCSV" → src/importers/csv-handler.ts, src/tests/csv.test.ts
     Read src/importers/csv-handler.ts → wraps parseCSV in validateAndParse()
     Grep "validateAndParse" → find consumers of the wrapper
     ```

  This pattern is essential for refactoring tasks (structured data extraction scenario): before modifying a utility, you must understand its full dependency graph to avoid breaking consumers.
