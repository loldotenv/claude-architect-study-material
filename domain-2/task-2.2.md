# Task 2.2: Implement structured error responses for MCP tools

## Knowledge of:

### ◇ The MCP `isError` flag pattern for communicating tool failures back to the agent

MCP tools use `isError: true` in the response to signal failure. The agent sees this and can decide how to handle it.

### ◇ The distinction between transient errors (timeouts, service unavailability), validation errors (invalid input), business errors (policy violations), and permission errors

Different error types need different handling:

| Error type | Example | Agent should... |
|---|---|---|
| Transient | Timeout, 503 | Retry |
| Validation | Invalid input format | Fix input and retry |
| Business | Policy violation | Explain to user, don't retry |
| Permission | Unauthorized | Escalate or inform user |

### ◇ Why uniform error responses (generic "Operation failed") prevent the agent from making appropriate recovery decisions

If every error just says "Operation failed," the agent can't distinguish between "retry might work" and "this will never work." It needs structured context to recover intelligently.

### ◇ The difference between retryable and non-retryable errors, and how returning structured metadata prevents wasted retry attempts

An `isRetryable: false` flag tells the agent not to waste turns retrying something that will never succeed.

## Skills in:

### ◆ Returning structured error metadata including `errorCategory` (transient/validation/permission), `isRetryable` boolean, and human-readable descriptions

Per the MCP spec, the `isError` flag lives at the result level alongside `content`:

```json
{
  "content": [
    {
      "type": "text",
      "text": "{\"errorCategory\": \"business\", \"isRetryable\": false, \"userMessage\": \"This account has been suspended. Please contact support.\"}"
    }
  ],
  "isError": true
}
```

The structured error metadata (category, retryability, user message) goes inside the `content` text, while `isError: true` signals the failure at the protocol level.

**MCP tool handler implementation (TypeScript SDK):**

```typescript
import { McpServer } from "@modelcontextprotocol/sdk/server/mcp.js";
import { z } from "zod";

const server = new McpServer({ name: "orders", version: "1.0.0" });

server.tool(
  "lookup_order",
  { order_id: z.string().regex(/^ORD-[A-Z0-9]{8}$/) },
  async ({ order_id }) => {
    try {
      const order = await db.orders.findById(order_id);

      if (!order) {
        // Valid empty result — NOT an error
        return {
          content: [{ type: "text", text: JSON.stringify({
            status: "not_found",
            message: `No order exists with ID ${order_id}`
          })}],
          isError: false  // This is a successful query with no match
        };
      }

      return {
        content: [{ type: "text", text: JSON.stringify(order) }],
        isError: false
      };

    } catch (err) {
      // Classify the error for the agent
      if (err.code === "ETIMEDOUT" || err.status === 503) {
        return {
          content: [{ type: "text", text: JSON.stringify({
            errorCategory: "transient",
            isRetryable: true,
            retryAfterMs: 2000,
            message: "Order service temporarily unavailable"
          })}],
          isError: true
        };
      }

      if (err.status === 403) {
        return {
          content: [{ type: "text", text: JSON.stringify({
            errorCategory: "permission",
            isRetryable: false,
            userMessage: "You don't have access to this order. It may belong to another account."
          })}],
          isError: true
        };
      }

      // Unknown error — non-retryable by default
      return {
        content: [{ type: "text", text: JSON.stringify({
          errorCategory: "unknown",
          isRetryable: false,
          message: err.message
        })}],
        isError: true
      };
    }
  }
);
```

Key patterns in this handler:
- **"Not found" is NOT an error** — it's a valid result (`isError: false`). The agent should tell the user, not retry.
- **Transient errors include `retryAfterMs`** — gives the agent a backoff hint.
- **Permission errors include `userMessage`** — the agent can relay this directly to the user.
- **Unknown errors default to non-retryable** — prevents infinite retry loops.

### ◆ Including `isRetryable: false` flags and customer-friendly explanations for business rule violations so the agent can communicate appropriately

Business errors need user-facing explanations, not just technical details.

### ◆ Implementing local error recovery within subagents for transient failures, propagating to the coordinator only errors that cannot be resolved locally along with partial results and what was attempted

Subagents should handle retries for transient failures internally. Only escalate to the coordinator when local recovery fails, including what was tried and any partial results.

### ◆ Distinguishing between access failures (needing retry decisions) and valid empty results (representing successful queries with no matches)

A timeout is different from "no results found." One needs a retry; the other is a valid answer. Your error structure must distinguish these. (This appears in exam Question 8.)
