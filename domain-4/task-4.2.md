# Task 4.2: Apply few-shot prompting to improve output consistency and quality

## Knowledge of:

### ◇ Few-shot examples as the most effective technique for achieving consistently formatted, actionable output when detailed instructions alone produce inconsistent results

Instructions describe what you want. Examples *show* what you want. When instructions aren't enough, add examples.

### ◇ The role of few-shot examples in demonstrating ambiguous-case handling (e.g., tool selection for ambiguous requests, branch-level test coverage gaps)

Examples are especially powerful for ambiguous cases where the "right" answer depends on judgment. Show Claude how you'd handle 2-3 ambiguous cases and it generalizes the pattern.

### ◇ How few-shot examples enable the model to generalize judgment to novel patterns rather than matching only pre-specified cases

Good examples teach the *reasoning pattern*, not just the specific cases shown. Claude applies the same judgment to new situations it hasn't seen in the examples.

### ◇ The effectiveness of few-shot examples for reducing hallucination in extraction tasks (e.g., handling informal measurements, varied document structures)

In extraction tasks, examples showing "this field is not in the document → return null" dramatically reduces fabrication compared to instructions alone.

## Skills in:

### ◆ Creating 2-4 targeted few-shot examples for ambiguous scenarios that show reasoning for why one action was chosen over plausible alternatives

Don't just show input→output. Show the *reasoning*: "This request mentions an order number, so we use `lookup_order` not `get_customer` because the customer is asking about order status, not account info."

### ◆ Including few-shot examples that demonstrate specific desired output format (location, issue, severity, suggested fix) to achieve consistency

Show the exact output structure you want repeated across all findings.

### ◆ Providing few-shot examples distinguishing acceptable code patterns from genuine issues to reduce false positives while enabling generalization

Show examples of code that *looks* like a bug but is actually fine (acceptable pattern), alongside genuine bugs. This teaches the boundary.

### ◆ Using few-shot examples to demonstrate correct handling of varied document structures (inline citations vs bibliographies, methodology sections vs embedded details)

Documents vary widely. Examples showing extraction from different structures help Claude handle novel formats.

### ◆ Adding few-shot examples showing correct extraction from documents with varied formats to address empty/null extraction of required fields

Show examples where a field is missing from the source document and the correct output is `null` — not a fabricated value.

---

### Code Example: Few-Shot for Handling Missing Fields

```python
import anthropic

client = anthropic.Anthropic()

few_shot_examples = """
<example>
<document>Invoice from TechCo, March 2024. Total: $500.</document>
<extraction>{"vendor_name": "TechCo", "invoice_number": null, "date": "2024-03-01", "total_amount": 500.00}</extraction>
<reasoning>No invoice number appears in the document — returned null rather than guessing.</reasoning>
</example>

<example>
<document>Receipt #R-443. Coffee Shop Express. Two lattes. Paid $12.50 cash.</document>
<extraction>{"vendor_name": "Coffee Shop Express", "invoice_number": "R-443", "date": null, "total_amount": 12.50}</extraction>
<reasoning>No date on this receipt — returned null. "R-443" used as invoice_number since it's the document identifier.</reasoning>
</example>
"""

message = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    messages=[{
        "role": "user",
        "content": (
            "Extract structured data from documents. Return null for fields "
            "not present — never fabricate values.\n\n"
            f"{few_shot_examples}\n"
            "<document>Acme Widget Co. Order total $2,340.00 USD.</document>\n"
            "Extract in the same JSON format."
        )
    }]
)
```

The few-shot examples teach Claude the **judgment pattern**: missing data becomes `null`, not a hallucinated value. This generalizes to novel document structures.
