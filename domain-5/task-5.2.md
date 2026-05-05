# Task 5.2: Design effective escalation and ambiguity resolution patterns

### Knowledge of:

- **Appropriate escalation triggers: customer requests for a human, policy exceptions/gaps (not just complex cases), and inability to make meaningful progress**

  Three clear triggers for escalation:
  1. Customer explicitly asks for a human
  2. Policy is ambiguous or doesn't cover the situation
  3. Agent can't make progress after reasonable attempts

- **The distinction between escalating immediately when a customer explicitly demands it versus offering to resolve when the issue is straightforward**

  - Customer says "Let me talk to a person" → escalate immediately, don't argue
  - Customer is frustrated but the issue is a simple return → acknowledge frustration, offer resolution, only escalate if they insist

- **Why sentiment-based escalation and self-reported confidence scores are unreliable proxies for actual case complexity**

  Sentiment does not equal complexity. A customer can be furious about a simple issue or calm about a complex one. Self-reported confidence is poorly calibrated — the agent may be "confident" about a wrong answer. (Exam Question 3: answer A, not B or D.)

- **How multiple customer matches require clarification (requesting additional identifiers) rather than heuristic selection**

  If `get_customer("John Smith")` returns 3 matches, ask for additional info (email, order number) — don't just pick the most recent one.

### Skills in:

- **Adding explicit escalation criteria with few-shot examples to the system prompt demonstrating when to escalate versus resolve autonomously**

  Show examples: "Customer asks about return policy for a standard item → resolve. Customer asks about competitor price matching when policy only covers own-site adjustments → escalate."

- **Honoring explicit customer requests for human agents immediately without first attempting investigation**

  "I want to speak to a human" → don't say "let me try to help you first." Just escalate.

- **Acknowledging frustration while offering resolution when the issue is within the agent's capability, escalating only if the customer reiterates their preference**

  "I understand this is frustrating. I can process this return right now — would you like me to proceed, or would you prefer to speak with a team member?"

- **Escalating when policy is ambiguous or silent on the customer's specific request (e.g., competitor price matching when policy only addresses own-site adjustments)**

  Don't guess about policy gaps. Escalate to someone who can make that call.

- **Instructing the agent to ask for additional identifiers when tool results return multiple matches, rather than selecting based on heuristics**

  "I found 3 accounts matching that name. Could you provide your email address or order number so I can pull up the right one?"
