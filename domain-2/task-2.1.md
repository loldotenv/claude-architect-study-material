# Task 2.1: Design effective tool interfaces with clear descriptions and boundaries

## Knowledge of:

### ◇ Tool descriptions as the primary mechanism LLMs use for tool selection; minimal descriptions lead to unreliable selection among similar tools

Claude does not "understand" tool purpose from the function name alone — it relies almost entirely on the `description` field to decide which tool to call. When two tools have similar names and minimal descriptions, selection becomes essentially random.

**Exam scenario — Customer Support Agent:**

Consider a support agent with both `get_customer` and `lookup_order`. If descriptions are:
- `get_customer`: "Gets customer information"
- `lookup_order`: "Looks up order information"

When a user asks "Where is my package?", Claude cannot reliably decide whether to start with the customer lookup (to get their order list) or the order lookup (if an order ID is already known). The description must encode the decision boundary.

**Bad vs Good — `lookup_order` example:**

```
BAD:
"Looks up order details."

GOOD:
"Retrieves order status, shipping info, and line items for a SINGLE known order.
 Input: order_id (string, format: ORD-XXXXXXXX).
 Use this when the customer has provided or you have already resolved a specific order ID.
 Do NOT use this to search for orders by customer name or email — use get_customer first
 to retrieve the customer's order list, then call this with a specific order_id.
 Returns: { status, tracking_number, items[], estimated_delivery }
 Edge cases: Returns isError if order_id format is invalid. Returns status 'not_found'
 for deleted/archived orders older than 2 years."
```

The good description encodes: input format, when to use vs. not use, relationship to sibling tools, return shape, and edge cases.

### ◇ The importance of including input formats, example queries, edge cases, and boundary explanations in tool descriptions

A good tool description is a mini-contract that answers five questions:

1. **What does it do?** — One-sentence purpose
2. **What inputs does it expect?** — Types, formats, constraints (e.g., "ISO-8601 date string")
3. **When should you use this vs. similar tools?** — Explicit disambiguation
4. **What does it return?** — Shape of successful response
5. **What edge cases exist?** — Empty results, invalid inputs, rate limits

Omitting any of these forces Claude to guess, which degrades reliability at scale. In a CI/CD scenario, a deploy tool missing edge-case documentation may cause Claude to retry deployments that failed due to a non-retryable config error.

### ◇ How ambiguous or overlapping tool descriptions cause misrouting (e.g., `analyze_content` vs `analyze_document` with near-identical descriptions)

If two tools have similar names and similar descriptions, Claude can't reliably distinguish them. This is Question 2 on the sample exam — the fix is better descriptions (answer B), not few-shot examples or routing layers.

### ◇ The impact of system prompt wording on tool selection: keyword-sensitive instructions can create unintended tool associations

Claude pattern-matches keywords in the system prompt against tool names and descriptions. This creates unintended "magnets" that pull tool selection toward keyword-matching tools regardless of context.

**How this happens — example:**

```
System prompt: "When the user asks about their account, always look up their
information before responding."

Available tools:
- lookup_order (description: "Looks up order details by order ID")
- get_customer (description: "Retrieves customer account information")
```

The phrase "look up" in the system prompt creates an association with `lookup_order` (because "look up" ≈ "lookup"). Even when the user asks a general account question that should route to `get_customer`, Claude may call `lookup_order` because the system prompt keyword "look up" primes that tool.

**Fix:** Use neutral verbs in system prompts ("retrieve the user's information" instead of "look up"), or reference tools by explicit name: "call `get_customer` when you need account details."

**Audit checklist for keyword interference:**
- Search your system prompt for verbs that match tool names (look up, analyze, search, fetch)
- Check for nouns that overlap with tool descriptions (document, content, data)
- Replace ambiguous keywords with explicit tool-name references where intent is clear

## Skills in:

### ◆ Writing tool descriptions that clearly differentiate each tool's purpose, expected inputs, outputs, and when to use it versus similar alternatives

Include: purpose, input format, output format, explicit "use this when..." / "don't use this when..." boundaries.

### ◆ Renaming tools and updating descriptions to eliminate functional overlap (e.g., renaming `analyze_content` to `extract_web_results` with a web-specific description)

Sometimes the fix is just renaming the tool to make its purpose unambiguous.

### ◆ Splitting generic tools into purpose-specific tools with defined input/output contracts (e.g., splitting `analyze_document` into `extract_data_points`, `summarize_content`, and `verify_claim_against_source`)

One vague tool → multiple specific tools with clear contracts. Reduces confusion.

### ◆ Reviewing system prompts for keyword-sensitive instructions that might override well-written tool descriptions

Even perfectly written tool descriptions can be overridden by system prompt wording. A practical review process:

1. **Extract all action verbs** from your system prompt (look up, analyze, search, fetch, check, verify)
2. **Cross-reference** each verb against your tool names and descriptions
3. **Flag overlaps** — any verb that appears in both the system prompt instruction and a tool name/description is a potential misrouting trigger
4. **Rewrite** flagged instructions to either use neutral language or explicitly name the intended tool

**Example fix in a customer support scenario:**

```
BEFORE (causes misrouting):
"Always search for relevant information before responding to the customer."
→ Claude calls `search_knowledge_base` even when user provides an order ID
   (should call `lookup_order` directly)

AFTER (explicit routing):
"If the customer provides an order ID, call `lookup_order`. If they have a
 general question without specific identifiers, call `search_knowledge_base`."
```
