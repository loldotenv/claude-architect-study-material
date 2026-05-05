# Task 1.7: Manage session state, resumption, and forking

## Knowledge of:

### ◇ Named session resumption using `--resume <session-name>` to continue a specific prior conversation

Named sessions let you pick up where you left off. Useful for multi-day investigation tasks or long-running projects where you want to maintain continuity.

```bash
# Resume a named session (or use the session picker)
claude --resume my-auth-refactor
# Short form
claude -r my-auth-refactor
```

You can also name or rename sessions with `/session rename` from within an active session.

### ◇ Session forking (`/fork` command or `fork` option in the SDK) for creating independent branches from a shared analysis baseline to explore divergent approaches

Example: after analyzing a codebase, fork into two sessions -- one to try testing strategy A, another for strategy B -- without them interfering with each other.

Each fork gets a copy of the context up to the fork point, then diverges independently. Note: forking branches conversation history, not the filesystem -- file changes in a fork are real and visible to other sessions in the same directory.

### ◇ The importance of informing the agent about changes to previously analyzed files when resuming sessions after code modifications

If files changed between sessions, tell the agent what changed. Otherwise it's working with stale information from prior `Read` tool calls that returned old file contents.

### ◇ Why starting a new session with a structured summary is more reliable than resuming with stale tool results

Resumed sessions may contain outdated file contents from prior `Read` calls. A fresh session with an injected summary of key findings avoids this staleness problem entirely.

Stale data risks:
- Agent makes recommendations based on code that no longer exists
- Agent skips re-reading files it "already read" (but contents changed)
- Agent's mental model diverges from reality

## Skills in:

### ◆ Using `--resume` with session names to continue named investigation sessions across work sessions

```bash
# Day 1: Start investigating auth system (name it during or after with /session rename)
claude "Analyze the authentication flow in src/auth/"

# Day 2: Resume by name
claude --resume auth-investigation "Now look at the session management piece"
```

### ◆ Using `/fork` (or the SDK `fork` option) to create parallel exploration branches (e.g., comparing two testing strategies or refactoring approaches from a shared codebase analysis)

Workflow:
1. Shared exploration phase: analyze codebase, identify issues
2. Fork at decision point
3. Branch A: try approach 1
4. Branch B: try approach 2
5. Compare results from both branches

### ◆ Choosing between session resumption (when prior context is mostly valid) and starting fresh with injected summaries (when prior tool results are stale)

| Situation | Approach |
|---|---|
| Files haven't changed, continuing same task | `--resume` |
| Files changed significantly, or tool results are outdated | New session with summary |
| Switching to a different aspect of the same project | New session with summary of relevant findings |
| Quick follow-up question on same topic | `--resume` |

### ◆ Informing a resumed session about specific file changes for targeted re-analysis rather than requiring full re-exploration

When resuming after changes, be specific:

```
"Since our last session, I refactored auth.ts:
- Extracted token validation into a separate validateToken() function
- Added a new middleware checkPermissions() in middleware.ts
- Removed the deprecated session.js file

Please re-analyze the auth flow with these changes in mind."
```

This enables targeted re-analysis instead of wasteful full re-exploration.
