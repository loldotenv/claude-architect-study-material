# Task 3.6: Integrate Claude Code into CI/CD pipelines

## Knowledge of:

### ◇ The `-p` (or `--print`) flag for running Claude Code in non-interactive mode in automated pipelines

Claude Code hangs in CI without `-p` because it waits for interactive input. `-p` processes the prompt and exits.

Exam Question 10: pipeline hangs -> add `-p` flag (answer A).

### ◇ `--output-format json` and `--json-schema` CLI flags for enforcing structured output in CI contexts

Machine-parseable output for automated posting as PR comments, CI annotations, etc. The JSON output includes metadata (session ID, token usage, costs) alongside the response. When `--json-schema` is provided, the structured output conforming to the schema appears in the `structured_output` field.

### ◇ CLAUDE.md as the mechanism for providing project context (testing standards, fixture conventions, review criteria) to CI-invoked Claude Code

CI-invoked Claude Code reads the same CLAUDE.md as interactive sessions. Put testing standards, review criteria, and fixture conventions there.

### ◇ Session context isolation: why the same Claude session that generated code is less effective at reviewing its own changes compared to an independent review instance

Self-review bias: the generating session retains its reasoning context and is less likely to question its own decisions. Use a separate, independent session for review.

## Skills in:

### ◆ Running Claude Code in CI with the `-p` flag to prevent interactive input hangs

```bash
claude -p "Analyze this pull request for security issues"
```

### ◆ Using `--output-format json` with `--json-schema` to produce machine-parseable structured findings for automated posting as inline PR comments

```bash
claude -p "Review this PR" --output-format json --json-schema schema.json
```

### ◆ Including prior review findings in context when re-running reviews after new commits, instructing Claude to report only new or still-unaddressed issues to avoid duplicate comments

Pass prior findings so Claude doesn't re-report already-addressed issues.

### ◆ Providing existing test files in context so test generation avoids suggesting duplicate scenarios already covered by the test suite

Include existing tests in context -> Claude generates complementary tests, not duplicates.

### ◆ Documenting testing standards, valuable test criteria, and available fixtures in CLAUDE.md to improve test generation quality and reduce low-value test output

Tell Claude what good tests look like in your project. What frameworks? What fixtures? What patterns?

In an automated PR review pipeline, combine `-p` with `--output-format json` and prior findings context to build a CI job that posts only new issues as inline PR comments — this is the exam's core CI/CD scenario, tying together non-interactive mode, structured output, session isolation (separate review instance from the code-generation session), and CLAUDE.md-driven review criteria.
