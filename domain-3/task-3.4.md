# Task 3.4: Determine when to use plan mode vs direct execution

## Knowledge of:

### ◇ Plan mode is designed for complex tasks involving large-scale changes, multiple valid approaches, and multi-file modifications

Plan mode lets Claude explore, analyze, and design before committing to changes. It's a "look before you leap" approach.

### ◇ Direct execution is appropriate for simple, well-scoped changes (e.g., adding a single validation check to one function)

If the task is clear, the scope is small, and there's only one reasonable approach — just do it.

### ◇ Plan mode enables safe codebase exploration and design before committing to changes, preventing costly rework

Exam Question 5: monolith-to-microservices restructuring -> plan mode (answer A). You need to understand dependencies before making changes.

### ◇ The Explore subagent for isolating verbose discovery output and returning summaries to preserve main conversation context

The Explore subagent does the heavy reading and returns a summary. Keeps the main context window clean.

## Skills in:

### ◆ Selecting plan mode for tasks with architectural implications (e.g., microservice restructuring, library migrations affecting 45+ files, choosing between integration approaches with different infrastructure requirements)

If the task has multiple valid approaches or affects many files, plan first.

### ◆ Selecting direct execution for well-understood changes with clear scope (e.g., a single-file bug fix with a clear stack trace, adding a date validation conditional)

If you can describe the fix in one sentence, direct execution is fine.

### ◆ Using the Explore subagent for verbose discovery phases to prevent context window exhaustion during multi-phase tasks

Let the Explore subagent do the reading; it returns a summary rather than filling your context with raw file contents.

### ◆ Combining plan mode for investigation with direct execution for implementation (e.g., planning a library migration, then executing the planned approach)

Plan mode to figure out what to do -> direct execution to do it.
