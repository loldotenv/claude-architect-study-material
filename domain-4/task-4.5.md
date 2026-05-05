# Task 4.5: Design efficient batch processing strategies

## Knowledge of:

### ◇ The Message Batches API: 50% cost savings, up to 24-hour processing window, no guaranteed latency SLA

Half the cost, but no latency guarantee. Results may take up to 24 hours.

### ◇ Batch processing is appropriate for non-blocking, latency-tolerant workloads (overnight reports, weekly audits, nightly test generation) and inappropriate for blocking workflows (pre-merge checks)

| Workflow | API | Why |
|---|---|---|
| Pre-merge checks | Synchronous | Developers are waiting |
| Nightly test generation | Batch | Can process overnight |
| Weekly audit reports | Batch | No urgency |

Exam Question 11: batch for overnight reports, real-time for pre-merge checks (answer A).

### ◇ The batch API does not support multi-turn tool calling within a single request (cannot execute tools mid-request and return results)

Batch requests are single-turn. No agentic loops within a batch request.

### ◇ `custom_id` fields for correlating batch request/response pairs

Each request has a `custom_id` that appears in the response, letting you match results to inputs.

## Skills in:

### ◆ Matching API approach to workflow latency requirements: synchronous API for blocking pre-merge checks, batch API for overnight/weekly analysis

Choose the API based on whether someone is waiting for the result.

### ◆ Calculating batch submission frequency based on SLA constraints (e.g., 4-hour windows to guarantee 30-hour SLA with 24-hour batch processing)

If batch processing takes up to 24 hours and you need results within 30 hours, submit at least 6 hours before the deadline.

### ◆ Handling batch failures: resubmitting only failed documents (identified by `custom_id`) with appropriate modifications (e.g., chunking documents that exceeded context limits)

Don't resubmit the entire batch. Identify failures by `custom_id`, fix the issue (e.g., chunk oversized docs), resubmit only those.

### ◆ Using prompt refinement on a sample set before batch-processing large volumes to maximize first-pass success rates and reduce iterative resubmission costs

Test your prompt on 10-20 samples first. Fix issues. Then batch-process thousands. Much cheaper than iterating on the full batch.

---

### Code Example: Batch Processing Thousands of Documents

```python
import anthropic
import json

client = anthropic.Anthropic()

# Scenario: Extract structured data from 5,000 invoices overnight (50% cost savings)

documents = [
    {"id": "inv-001", "text": "Invoice #1001 from Acme Corp..."},
    {"id": "inv-002", "text": "Receipt from Bob's Hardware..."},
    # ... thousands more
]

extract_tool = {
    "name": "extract_invoice",
    "description": "Extract structured invoice data.",
    "input_schema": {
        "type": "object",
        "properties": {
            "vendor_name": {"type": "string"},
            "total_amount": {"type": "number"},
            "date": {"type": ["string", "null"]},
        },
        "required": ["vendor_name", "total_amount"]
    }
}

# 1. Build batch requests with custom_id for correlation
requests = []
for doc in documents:
    requests.append({
        "custom_id": doc["id"],  # Links response back to input document
        "params": {
            "model": "claude-sonnet-4-20250514",
            "max_tokens": 1024,
            "tools": [extract_tool],
            "tool_choice": {"type": "tool", "name": "extract_invoice"},
            "messages": [{
                "role": "user",
                "content": f"Extract structured data:\n\n{doc['text']}"
            }]
        }
    })

# 2. Submit the batch
batch = client.messages.batches.create(requests=requests)
print(f"Batch submitted: {batch.id}, status: {batch.processing_status}")

# 3. Poll for completion (up to 24 hours)
import time
while True:
    batch = client.messages.batches.retrieve(batch.id)
    if batch.processing_status == "ended":
        break
    time.sleep(300)  # Check every 5 minutes

# 4. Process results, matching by custom_id
failed_ids = []
for result in client.messages.batches.results(batch.id):
    if result.result.type == "succeeded":
        message = result.result.message
        tool_block = next(b for b in message.content if b.type == "tool_use")
        extracted = tool_block.input
        save_to_database(result.custom_id, extracted)
    else:
        failed_ids.append(result.custom_id)

# 5. Resubmit only failed documents (e.g., chunk oversized ones)
if failed_ids:
    retry_requests = build_retry_requests(failed_ids)  # Fix issues first
    client.messages.batches.create(requests=retry_requests)
```

Key points for the exam:
- **`custom_id`** is how you correlate responses to input documents — essential for batch workflows
- Batch API does **not** support multi-turn tool calling (no agentic loops)
- Use batch for overnight/weekly workloads; use synchronous API for pre-merge checks where developers are waiting
- Test prompts on a sample set before submitting thousands to avoid costly resubmissions
