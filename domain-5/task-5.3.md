# Task 5.3: Implement error propagation strategies across multi-agent systems

### Knowledge of:

- **Structured error context (failure type, attempted query, partial results, alternative approaches) as enabling intelligent coordinator recovery decisions**

  The coordinator needs to know:
  - What failed (failure type)
  - What was attempted (query/parameters)
  - What partial results were obtained
  - What alternatives exist

  With this context, it can make intelligent recovery decisions.

- **The distinction between access failures (timeouts needing retry decisions) and valid empty results (successful queries with no matches)**

  | Response | Meaning | Action |
  |---|---|---|
  | Timeout / 503 | Access failure | Retry or try alternative |
  | Empty result set | Valid — no matches found | Proceed with "no data found" |

  These look similar but require completely different handling.

- **Why generic error statuses ("search unavailable") hide valuable context from the coordinator**

  "Search unavailable" tells the coordinator nothing useful. Was it a timeout? A rate limit? A bad query? The coordinator can't choose the right recovery strategy without details. (Exam Question 8: answer A.)

- **Why silently suppressing errors (returning empty results as success) or terminating entire workflows on single failures are both anti-patterns**

  - Suppressing errors → coordinator proceeds with incomplete data, doesn't know coverage is lacking
  - Terminating on first failure → throws away partial results from other subagents that succeeded

### Skills in:

- **Returning structured error context including failure type, what was attempted, partial results, and potential alternatives to enable coordinator recovery**

  ```json
  {
    "status": "partial_failure",
    "failure_type": "timeout",
    "attempted_query": "AI impact on music industry",
    "partial_results": ["Found 2 of 5 expected sources"],
    "alternatives": ["Try narrower query", "Use cached results from previous run"]
  }
  ```

- **Distinguishing access failures from valid empty results in error reporting so the coordinator can make appropriate decisions**

  The error structure must clearly differentiate "I couldn't reach the service" from "I reached it and found nothing."

- **Having subagents implement local recovery for transient failures and only propagate errors they cannot resolve, including what was attempted and partial results**

  Subagents should retry transient failures internally (2-3 attempts). Only escalate to the coordinator when local recovery fails.

- **Structuring synthesis output with coverage annotations indicating which findings are well-supported versus which topic areas have gaps due to unavailable sources**

  The final report should note: "Findings on visual arts are well-supported (5 sources). Music industry coverage is limited (1 source due to search timeout)."
