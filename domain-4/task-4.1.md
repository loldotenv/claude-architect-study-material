# Task 4.1: Design prompts with explicit criteria to improve precision and reduce false positives

### Knowledge of:

- **The importance of explicit criteria over vague instructions (e.g., "flag comments only when claimed behavior contradicts actual code behavior" vs "check that comments are accurate")**

  Vague: "check that comments are accurate" — what does "accurate" mean?
  Explicit: "flag comments only when the described behavior contradicts the actual code behavior" — clear, testable criteria.

- **How general instructions like "be conservative" or "only report high-confidence findings" fail to improve precision compared to specific categorical criteria**

  "Be conservative" is subjective — Claude's interpretation varies. Specific criteria like "only report: security vulnerabilities, null reference risks, and API contract violations" gives a clear checklist.

- **The impact of false positive rates on developer trust: high false positive categories undermine confidence in accurate categories**

  If 50% of style findings are wrong, developers start ignoring ALL findings — including the valid security ones.

### Skills in:

- **Writing specific review criteria that define which issues to report (bugs, security) versus skip (minor style, local patterns) rather than relying on confidence-based filtering**

  Define categories explicitly: "Report: security issues, logic errors, null references. Skip: style preferences, naming conventions, comment formatting."

- **Temporarily disabling high false-positive categories to restore developer trust while improving prompts for those categories**

  If "unused variable" findings are 60% wrong, disable that category entirely while you fix the prompt. Re-enable once precision improves.

- **Defining explicit severity criteria with concrete code examples for each severity level to achieve consistent classification**

  ```
  CRITICAL: SQL injection, authentication bypass
  HIGH: Null reference in production path, race condition
  MEDIUM: Missing error handling for external API calls
  LOW: Unused imports, redundant type assertions
  ```
