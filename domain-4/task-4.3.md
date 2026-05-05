# Task 4.3: Enforce structured output using tool use and JSON schemas

## Knowledge of:

### ◇ Tool use (`tool_use`) with JSON schemas as the most reliable approach for guaranteed schema-compliant structured output, eliminating JSON syntax errors

`tool_use` forces Claude to output valid JSON matching your schema. No parsing errors, no malformed JSON.

### ◇ The distinction between `tool_choice: "auto"` (model may return text instead of calling a tool), `"any"` (model must call a tool but can choose which), and forced tool selection (model must call a specific named tool)

| Setting | Claude can return text? | Claude chooses the tool? |
|---|---|---|
| `"auto"` | Yes | Yes |
| `"any"` | No | Yes (from available tools) |
| `{"type": "tool", "name": "X"}` | No | No (must call X) |

Note: with `"any"` or forced selection, Claude will not emit natural language before the tool use block, even if instructed to.

### ◇ That strict JSON schemas via tool use eliminate syntax errors but do not prevent semantic errors (e.g., line items that don't sum to total, values in wrong fields)

Schema validation catches structure problems (missing fields, wrong types). It does NOT catch logic problems (amounts that don't add up, dates in the wrong field). You need separate validation for semantic correctness.

### ◇ Schema design considerations: required vs optional fields, enum fields with "other" + detail string patterns for extensible categories

- Make fields optional/nullable when the source might not contain the information — prevents hallucination to satisfy required fields
- Use enum + "other" pattern for categories: `["invoice", "receipt", "contract", "other"]` with an `other_detail` string field

## Skills in:

### ◆ Defining extraction tools with JSON schemas as input parameters and extracting structured data from the `tool_use` response

Define a tool whose input schema IS your desired output format. Claude "calls" the tool with the extracted data.

### ◆ Setting `tool_choice: "any"` to guarantee structured output when multiple extraction schemas exist and the document type is unknown

Let Claude pick the right extraction schema while guaranteeing it produces structured output.

### ◆ Forcing a specific tool with `tool_choice: {"type": "tool", "name": "extract_metadata"}` to ensure a particular extraction runs before enrichment steps

Use forced selection when step ordering matters.

### ◆ Designing schema fields as optional (nullable) when source documents may not contain the information, preventing the model from fabricating values to satisfy required fields

If a field is required, Claude will fill it even if the data doesn't exist in the source — leading to hallucination. Make it nullable.

### ◆ Adding enum values like `"unclear"` for ambiguous cases and `"other"` + detail fields for extensible categorization

Give Claude an escape hatch for ambiguous cases. `"unclear"` is better than forcing a wrong category.

### ◆ Including format normalization rules in prompts alongside strict output schemas to handle inconsistent source formatting

The schema enforces structure; the prompt handles format normalization (e.g., "normalize all dates to ISO 8601").

---

### Complete Code Example: Structured Invoice Extraction

```python
import anthropic
import json

client = anthropic.Anthropic()

# 1. Define an extraction tool with a JSON schema
extract_invoice_tool = {
    "name": "extract_invoice",
    "description": "Extract structured data from an invoice or receipt.",
    "input_schema": {
        "type": "object",
        "properties": {
            "vendor_name": {
                "type": "string",
                "description": "Name of the vendor/company on the invoice"
            },
            "invoice_number": {
                "type": ["string", "null"],
                "description": "Invoice number if present, null if not found"
            },
            "date": {
                "type": ["string", "null"],
                "description": "Invoice date in ISO 8601 format (YYYY-MM-DD)"
            },
            "total_amount": {
                "type": "number",
                "description": "Total amount due"
            },
            "currency": {
                "type": "string",
                "enum": ["USD", "EUR", "GBP", "CAD", "other"],
                "description": "Currency of the invoice"
            },
            "currency_other_detail": {
                "type": ["string", "null"],
                "description": "If currency is 'other', specify the actual currency"
            },
            "document_type": {
                "type": "string",
                "enum": ["invoice", "receipt", "credit_note", "other"]
            },
            "line_items": {
                "type": "array",
                "items": {
                    "type": "object",
                    "properties": {
                        "description": {"type": "string"},
                        "quantity": {"type": ["number", "null"]},
                        "unit_price": {"type": ["number", "null"]},
                        "amount": {"type": "number"}
                    },
                    "required": ["description", "amount"]
                }
            }
        },
        "required": ["vendor_name", "total_amount", "currency", "document_type"]
    }
}

# 2. Call the API with forced tool selection
document_text = """
INVOICE #2024-0892
Acme Corp | Date: March 15, 2024

Items:
- Widget A (x10) .......... $45.00
- Service Fee .............. $120.00

Total: $165.00 USD
"""

message = client.messages.create(
    model="claude-sonnet-4-20250514",
    max_tokens=1024,
    tools=[extract_invoice_tool],
    tool_choice={"type": "tool", "name": "extract_invoice"},  # Force this tool
    messages=[{
        "role": "user",
        "content": f"Extract structured data from this document:\n\n{document_text}"
    }]
)

# 3. Extract structured data from the tool_use response
tool_use_block = next(b for b in message.content if b.type == "tool_use")
invoice_data = tool_use_block.input

print(json.dumps(invoice_data, indent=2))
# {
#   "vendor_name": "Acme Corp",
#   "invoice_number": "2024-0892",
#   "date": "2024-03-15",
#   "total_amount": 165.00,
#   "currency": "USD",
#   "currency_other_detail": null,
#   "document_type": "invoice",
#   "line_items": [
#     {"description": "Widget A", "quantity": 10, "unit_price": 4.50, "amount": 45.00},
#     {"description": "Service Fee", "quantity": null, "unit_price": null, "amount": 120.00}
#   ]
# }
```

Key design choices demonstrated:
- **`invoice_number` and `date` are nullable** — prevents hallucination when fields are absent
- **`currency` uses enum + "other" pattern** — extensible without breaking schema
- **`line_items` sub-objects have optional `quantity`/`unit_price`** — handles service fees with no unit breakdown
- **`tool_choice` forces the specific tool** — guarantees structured output, no text preamble
