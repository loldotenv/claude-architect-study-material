# Task 5.4: Manage context effectively in large codebase exploration

## Knowledge of:

### ◇ Context degradation in extended sessions: models start giving inconsistent answers and referencing "typical patterns" rather than specific classes discovered earlier

After many turns, the model's recall of early findings degrades. It starts hallucinating based on general knowledge instead of citing specific classes/functions it read earlier.

### ◇ The role of scratchpad files for persisting key findings across context boundaries

Write key findings to a scratchpad file as you discover them. Reference the file in later prompts instead of relying on context memory.

### ◇ Subagent delegation for isolating verbose exploration output while the main agent coordinates high-level understanding

Let subagents do the deep file reading. They return summaries. The main agent stays focused on coordination without its context being filled with raw file contents.

### ◇ Structured state persistence for crash recovery: each agent exports state to a known location, and the coordinator loads a manifest on resume

If the system crashes mid-exploration, structured state exports let you resume from where you left off rather than starting over.

## Skills in:

### ◆ Spawning subagents to investigate specific questions (e.g., "find all test files," "trace refund flow dependencies") while the main agent preserves high-level coordination

The main agent thinks about *what to investigate*. Subagents do the *investigation*. This preserves the main agent's context for coordination.

### ◆ Having agents maintain scratchpad files recording key findings, referencing them for subsequent questions to counteract context degradation

```
# scratchpad.md
- Auth module: src/auth/index.ts, exports: login(), logout(), refreshToken()
- Refund flow: src/billing/refund.ts → calls src/payments/stripe.ts
- Test coverage gap: no tests for refreshToken()
```

### ◆ Summarizing key findings from one exploration phase before spawning subagents for the next phase, injecting summaries into initial context

Phase 1 → summarize findings → inject summary into Phase 2 agent prompts.

### ◆ Designing crash recovery using structured agent state exports (manifests) that the coordinator loads on resume and injects into agent prompts

Each agent periodically exports its state. On crash, the coordinator reads the manifests and resumes with injected state.

### ◆ Using `/compact` to reduce context usage during extended exploration sessions when context fills with verbose discovery output

`/compact` summarizes the conversation history to free up context space. Use it when exploration output is filling the context window. Note: after compaction, the project-root CLAUDE.md is re-read and re-injected automatically; nested subdirectory CLAUDE.md files reload the next time Claude reads a file in that subdirectory.

In the developer productivity scenario (exploring an unfamiliar monorepo), these techniques combine: use subagents to trace dependency graphs across packages, persist findings in a scratchpad so onboarding developers build a mental model progressively, and apply `/compact` when the exploration fills context — this mirrors how Task 3.1's directory-level CLAUDE.md files provide package-specific guidance that aids targeted exploration.
