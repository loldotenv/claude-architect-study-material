# Task 4.6: Design multi-instance and multi-pass review architectures

### Knowledge of:

- **Self-review limitations: a model retains reasoning context from generation, making it less likely to question its own decisions in the same session**

  The same session that wrote the code remembers *why* it made each decision — making it biased toward approving its own work. This is why self-review is unreliable.

- **Independent review instances (without prior reasoning context) are more effective at catching subtle issues than self-review instructions or extended thinking**

  A fresh Claude instance reviewing code without the generation context catches issues the generator would rationalize away.

- **Multi-pass review: splitting large reviews into per-file local analysis passes plus cross-file integration passes to avoid attention dilution and contradictory findings**

  Exam Question 12: 14-file PR with inconsistent review quality → split into per-file passes + integration pass (answer A).

  Why single-pass fails on large PRs:
  - Attention dilutes across many files
  - Later files get less thorough review
  - Cross-file contradictions (flagging a pattern in one file, approving it in another)

### Skills in:

- **Using a second independent Claude instance to review generated code without the generator's reasoning context**

  Generate with session A. Review with session B (no shared context). Session B sees only the code, not the reasoning.

- **Splitting large multi-file reviews into focused per-file passes for local issues plus separate integration passes for cross-file data flow analysis**

  1. Per-file passes: analyze each file individually for local issues
  2. Integration pass: examine cross-file data flow, consistency, API contracts

- **Running verification passes where the model self-reports confidence alongside each finding to enable calibrated review routing**

  Have Claude rate its confidence per finding. Route low-confidence findings to human review; auto-apply high-confidence ones.
