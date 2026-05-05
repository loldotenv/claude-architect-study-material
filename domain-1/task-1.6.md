# Task 1.6: Design task decomposition strategies for complex workflows

### Knowledge of:

- **When to use fixed sequential pipelines (prompt chaining) versus dynamic adaptive decomposition based on intermediate findings**

  | Pattern | Best for | Example |
  |---|---|---|
  | Prompt chaining (fixed) | Predictable, well-defined steps | Code review: analyze each file -> cross-file integration pass |
  | Dynamic decomposition | Open-ended, discovery-driven tasks | "Add comprehensive tests to a legacy codebase" |

  Key distinction: if you can enumerate all steps upfront, use chaining. If the next step depends on what you discover, use dynamic decomposition.

- **Prompt chaining patterns that break reviews into sequential steps (e.g., analyze each file individually, then run a cross-file integration pass)**

  For code review with multiple files:
  1. **Per-file local analysis**: Each file is reviewed independently for local issues (bugs, style, logic errors)
  2. **Cross-file integration pass**: A separate pass examines how files interact (API contracts, shared state, dependency issues)

  This avoids attention dilution -- reviewing 10 files simultaneously means each gets less focus than reviewing them one at a time.

- **The value of adaptive investigation plans that generate subtasks based on what is discovered at each step**

  You can't plan everything upfront for open-ended tasks. The plan must evolve:
  - Step 1 might reveal that a dependency needs updating first
  - Step 2 might reveal that two modules are tightly coupled
  - Step 3 adapts based on what steps 1 and 2 found

### Skills in:

- **Selecting task decomposition patterns appropriate to the workflow: prompt chaining for predictable multi-aspect reviews, dynamic decomposition for open-ended investigation tasks**

  Match the pattern to the task's predictability:
  - Known steps, known order -> Prompt chaining
  - Known goal, unknown path -> Dynamic decomposition
  - Known steps, unknown order -> Coordinator with parallel subagents

- **Splitting large code reviews into per-file local analysis passes plus a separate cross-file integration pass to avoid attention dilution**

  This is the answer to exam Question 12.

  ```
  Phase 1 (parallel, per-file):
    - Review file_a.py for local issues
    - Review file_b.py for local issues
    - Review file_c.py for local issues

  Phase 2 (sequential, cross-file):
    - Check API contracts between file_a and file_b
    - Check shared state management across all files
    - Check for inconsistent error handling patterns
  ```

  Why this works: each per-file pass gets full attention on one file. The integration pass then focuses specifically on inter-file concerns.

- **Decomposing open-ended tasks (e.g., "add comprehensive tests to a legacy codebase") by first mapping structure, identifying high-impact areas, then creating a prioritized plan that adapts as dependencies are discovered**

  Recommended approach:
  1. **Map the codebase structure** -- understand modules, entry points, dependencies
  2. **Identify high-impact areas** -- most critical functionality, least test coverage
  3. **Create a prioritized plan** -- start with highest impact, lowest complexity
  4. **Adapt as you discover dependencies** -- if module A depends on module B, test B first

  This is dynamic decomposition in practice: the plan evolves based on what each step reveals.
