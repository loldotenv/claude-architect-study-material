# Task 5.1: Manage conversation context to preserve critical information across long interactions

### Knowledge of:

- **Progressive summarization risks: condensing numerical values, percentages, dates, and customer-stated expectations into vague summaries**

  Summarization loses precision. "The customer was charged $47.99 on March 3rd" becomes "the customer was overcharged recently." The specific values that matter for resolution disappear.

- **The "lost in the middle" effect: models reliably process information at the beginning and end of long inputs but may omit findings from middle sections**

  Information in the middle of long contexts gets less attention. Critical facts should be placed at the beginning or end, or highlighted with explicit section headers.

- **How tool results accumulate in context and consume tokens disproportionately to their relevance (e.g., 40+ fields per order lookup when only 5 are relevant)**

  A single `lookup_order` call might return 40 fields, but you only need order ID, status, amount, date, and customer ID. The other 35 fields waste context space.

- **The importance of passing complete conversation history in subsequent API requests to maintain conversational coherence**

  Each API call needs the full conversation history. Truncating it causes Claude to lose context and behave inconsistently.

### Skills in:

- **Extracting transactional facts (amounts, dates, order numbers, statuses) into a persistent "case facts" block included in each prompt, outside summarized history**

  ```
  === CASE FACTS ===
  Customer: Alice (#12345)
  Order: #67890 - $47.99 - Delivered March 3
  Issue: Charged twice
  ==================
  ```
  This block persists even as conversation history gets summarized.

- **Extracting and persisting structured issue data (order IDs, amounts, statuses) into a separate context layer for multi-issue sessions**

  Separate structured data from conversational flow. The structured layer never gets summarized away.

- **Trimming verbose tool outputs to only relevant fields before they accumulate in context (e.g., keeping only return-relevant fields from order lookups)**

  Don't pass all 40 fields to Claude. Extract the 5 relevant ones and discard the rest before adding to context.

- **Placing key findings summaries at the beginning of aggregated inputs and organizing detailed results with explicit section headers to mitigate position effects**

  Summary first, details after. Use clear section headers so Claude can navigate the content.

- **Requiring subagents to include metadata (dates, source locations, methodological context) in structured outputs to support accurate downstream synthesis**

  Subagent outputs need metadata for the synthesis agent to produce accurate, attributed summaries.

- **Modifying upstream agents to return structured data (key facts, citations, relevance scores) instead of verbose content and reasoning chains when downstream agents have limited context budgets**

  Upstream agents should return concise structured data, not lengthy reasoning. The downstream agent doesn't need to see the reasoning — just the facts and their sources.
