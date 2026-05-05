# Task 4.4: Implement validation, retry, and feedback loops for extraction quality

### Knowledge of:

- **Retry-with-error-feedback: appending specific validation errors to the prompt on retry to guide the model toward correction**

  Don't just retry — tell Claude what went wrong. "The 'amount' field should be a number but you returned a string '45.99'. Please extract it as a numeric value."

- **The limits of retry: retries are ineffective when the required information is simply absent from the source document (vs format or structural errors)**

  | Error type | Retry effective? | Why |
  |---|---|---|
  | Format mismatch (string vs number) | Yes | Claude can fix the format |
  | Structural error (wrong field) | Yes | Claude can reorganize |
  | Information absent from source | No | Can't extract what doesn't exist |

- **Feedback loop design: tracking which code constructs trigger findings (`detected_pattern` field) to enable systematic analysis of dismissal patterns**

  Add a `detected_pattern` field to findings. When developers dismiss findings, you can analyze *which patterns* produce false positives and refine your prompts accordingly.

- **The difference between semantic validation errors (values don't sum, wrong field placement) and schema syntax errors (eliminated by tool use)**

  Tool use eliminates syntax errors automatically. You still need custom validation for semantic correctness.

### Skills in:

- **Implementing follow-up requests that include the original document, the failed extraction, and specific validation errors for model self-correction**

  On retry, include:
  1. Original document
  2. The failed extraction attempt
  3. Specific validation errors

  This gives Claude maximum context to self-correct.

- **Identifying when retries will be ineffective (e.g., information exists only in an external document not provided) versus when they will succeed (format mismatches, structural output errors)**

  If the information isn't in the source, retrying won't help. Return null/unknown instead.

- **Adding `detected_pattern` fields to structured findings to enable analysis of false positive patterns when developers dismiss findings**

  Track what triggered each finding so you can analyze and improve over time.

- **Designing self-correction validation flows: extracting "calculated_total" alongside "stated_total" to flag discrepancies, adding "conflict_detected" booleans for inconsistent source data**

  Have Claude extract both the calculated and stated values. If they don't match, flag it — don't silently pick one.

---

### Code Example: Retry-with-Feedback Pattern

```python
import anthropic
import json

client = anthropic.Anthropic()

def validate_extraction(data: dict) -> list[str]:
    """Return list of validation errors (empty = valid)."""
    errors = []
    if data.get("line_items"):
        calculated = sum(item["amount"] for item in data["line_items"])
        if abs(calculated - data["total_amount"]) > 0.01:
            errors.append(
                f"Line items sum to {calculated} but total_amount is "
                f"{data['total_amount']}. Re-check amounts."
            )
    if data.get("date") and not data["date"].startswith("20"):
        errors.append(
            f"Date '{data['date']}' is not in ISO 8601 format (YYYY-MM-DD)."
        )
    return errors

def extract_with_retry(document: str, tool: dict, max_retries: int = 2) -> dict:
    messages = [{
        "role": "user",
        "content": f"Extract structured data from this document:\n\n{document}"
    }]

    for attempt in range(max_retries + 1):
        response = client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=1024,
            tools=[tool],
            tool_choice={"type": "tool", "name": tool["name"]},
            messages=messages,
        )

        tool_block = next(b for b in response.content if b.type == "tool_use")
        extraction = tool_block.input
        errors = validate_extraction(extraction)

        if not errors:
            return extraction  # Valid — done

        if attempt == max_retries:
            return extraction  # Out of retries — return best effort

        # Append the failed attempt and errors as feedback for retry
        messages.append({"role": "assistant", "content": response.content})
        messages.append({
            "role": "user",
            "content": (
                "The extraction has validation errors. Please fix them:\n"
                + "\n".join(f"- {e}" for e in errors)
                + "\n\nRe-extract with corrections."
            )
        })

    return extraction
```

Key points:
- The conversation grows: original request -> failed attempt -> specific error feedback -> corrected attempt
- Validation errors are **specific and actionable** (not "try again")
- Limited retries prevent infinite loops when the source data is genuinely ambiguous
- Retries are effective for format/structural errors but not for absent information
