# Task 1.4: Implement multi-step workflows with enforcement and handoff patterns

### Knowledge of:

- **The difference between programmatic enforcement (hooks, prerequisite gates) and prompt-based guidance for workflow ordering**

  | Approach | Guarantee level | Use when |
  |---|---|---|
  | Programmatic (hooks/gates) | Deterministic -- 100% enforced | Financial operations, identity verification, compliance |
  | Prompt-based | Probabilistic -- usually followed | Soft preferences, general guidance |

  Key insight: prompt instructions have a non-zero failure rate. For most workflows this is fine. For security-critical steps, it is not.

- **When deterministic compliance is required (e.g., identity verification before financial operations), prompt instructions alone have a non-zero failure rate**

  If skipping a step has financial or security consequences, you cannot rely on prompt instructions alone. Even a 1% failure rate is unacceptable for operations like processing refunds without verifying customer identity.

  This is the core principle tested in exam Question 1.

- **Structured handoff protocols for mid-process escalation that include customer details, root cause analysis, and recommended actions**

  When escalating to a human, the handoff should be a structured summary -- not the raw conversation transcript. Human agents need:
  - Customer identification
  - Root cause analysis
  - What was already tried
  - Recommended next action
  - Relevant amounts/dates

### Skills in:

- **Implementing programmatic prerequisites that block downstream tool calls until prerequisite steps have completed (e.g., blocking `process_refund` until `get_customer` has returned a verified customer ID)**

  ```python
  # Programmatic gate - runs BEFORE the tool executes
  def pre_tool_hook(tool_name, tool_input, context):
      if tool_name == "process_refund":
          if not context.get("customer_verified"):
              return block("Must verify customer identity before processing refund")
      return allow()
  ```

  This is the exam's Question 1 answer: use a programmatic gate, not prompt instructions.

- **Decomposing multi-concern customer requests into distinct items, then investigating each in parallel using shared context before synthesizing a unified resolution**

  Customer: "I was charged twice and my item arrived damaged."

  1. **Decompose**: Issue A (double charge), Issue B (damaged item)
  2. **Investigate in parallel**: Look up billing records + look up shipping/delivery records
  3. **Synthesize**: Unified response addressing both issues with a single resolution plan

- **Compiling structured handoff summaries (customer ID, root cause, refund amount, recommended action) when escalating to human agents who lack access to the conversation transcript**

  ```json
  {
    "customer_id": "CUST-12345",
    "customer_name": "Jane Smith",
    "root_cause": "Duplicate charge due to payment gateway timeout",
    "amount_in_question": "$89.99",
    "actions_taken": ["Verified customer identity", "Confirmed duplicate in billing system"],
    "recommended_action": "Process refund of $89.99 to original payment method",
    "escalation_reason": "Refund exceeds automated threshold ($50)"
  }
  ```

  Human agents don't see the AI conversation. Give them everything they need in a structured format.
