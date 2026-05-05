# Task 1.5: Apply Agent SDK hooks for tool call interception and data normalization

## Knowledge of:

### ◇ Hook patterns (e.g., `PostToolUse`) that intercept tool results for transformation before the model processes them

Hooks sit between the tool execution and the model, letting you transform data before Claude sees it. The flow:

```
Tool executes -> PostToolUse hook transforms output -> Claude receives normalized result
```

### ◇ Hook patterns that intercept outgoing tool calls to enforce compliance rules (e.g., blocking refunds above a threshold)

Pre-execution hooks can inspect and block tool calls:

```
Claude requests tool call -> PreToolUse hook checks policy -> Allow or Block
```

If blocked, the hook returns an error message that Claude sees instead of the tool result, allowing it to take an alternative action.

### ◇ The distinction between using hooks for deterministic guarantees versus relying on prompt instructions for probabilistic compliance

This is a recurring exam theme:
- **Hooks** = deterministic. The code runs every time, no exceptions.
- **Prompts** = probabilistic. Claude usually follows them, but not 100%.

When the consequence of non-compliance is serious (financial, security), use hooks.

| Scenario | Use hooks? | Use prompts? |
|---|---|---|
| Block refunds over $500 | Yes | Optional (as additional guidance) |
| Prefer concise responses | No | Yes |
| Require identity verification before account changes | Yes | Optional |
| Suggest checking documentation first | No | Yes |

## Skills in:

### ◆ Implementing `PostToolUse` hooks to normalize heterogeneous data formats (Unix timestamps, ISO 8601, numeric status codes) from different MCP tools before the agent processes them

Different tools return dates in different formats. A hook can normalize them all to ISO 8601 before Claude sees them, preventing confusion.

**Actual Agent SDK registration pattern** (Python):

```python
import asyncio
from claude_agent_sdk import ClaudeSDKClient, ClaudeAgentOptions, HookMatcher

async def normalize_timestamps(input_data, tool_use_id, context):
    """PostToolUse hook — transform tool output before Claude sees it."""
    tool_output = input_data.get("tool_output", "")

    # Replace unix timestamps with ISO 8601 in the output
    normalized = convert_unix_to_iso(tool_output)

    # Replace status codes with human-readable labels
    normalized = replace_status_codes(normalized, STATUS_MAP)

    return {
        "hookSpecificOutput": {
            "hookEventName": input_data["hook_event_name"],
            "updatedToolOutput": normalized,
        }
    }

async def main():
    options = ClaudeAgentOptions(
        hooks={
            "PostToolUse": [
                HookMatcher(
                    matcher="^mcp__",  # Only normalize MCP tool outputs
                    hooks=[normalize_timestamps],
                )
            ]
        }
    )

    async with ClaudeSDKClient(options=options) as client:
        await client.query("Look up the customer's last 3 orders")
        async for message in client.receive_response():
            print(message)

asyncio.run(main())
```

Key points: hooks are registered via `ClaudeAgentOptions(hooks={...})` with event names as keys and lists of `HookMatcher` objects as values. Each `HookMatcher` has an optional `matcher` regex (filters by tool name) and a `hooks` list of async callbacks. PostToolUse callbacks return `updatedToolOutput` inside `hookSpecificOutput` to replace the output Claude sees.

### ◆ Implementing tool call interception hooks that block policy-violating actions (e.g., refunds exceeding $500) and redirect to alternative workflows (e.g., human escalation)

**Actual Agent SDK registration pattern** (Python):

```python
from claude_agent_sdk import ClaudeAgentOptions, HookMatcher

async def block_large_refunds(input_data, tool_use_id, context):
    """PreToolUse hook — block refunds over $500 and redirect to escalation."""
    amount = input_data["tool_input"].get("amount", 0)
    if amount > 500:
        return {
            "systemMessage": (
                "The refund was blocked by policy. Compile a handoff summary "
                "and inform the customer a supervisor will process it."
            ),
            "hookSpecificOutput": {
                "hookEventName": input_data["hook_event_name"],
                "permissionDecision": "deny",
                "permissionDecisionReason": (
                    f"Refund of ${amount} exceeds $500 limit. "
                    "Escalating to human supervisor."
                ),
            },
        }
    return {}

options = ClaudeAgentOptions(
    hooks={
        "PreToolUse": [
            HookMatcher(
                matcher="process_refund",
                hooks=[block_large_refunds],
            )
        ]
    }
)
```

The `matcher` regex ensures this hook only fires for the `process_refund` tool. Returning `permissionDecision: "deny"` is a deterministic block — Claude never executes the tool. The `systemMessage` injects guidance into the conversation so Claude knows to escalate.

### ◆ Choosing hooks over prompt-based enforcement when business rules require guaranteed compliance

Rule of thumb: if you'd be fired for a violation, use a hook, not a prompt.

Decision framework:
1. What is the consequence of non-compliance? (Minor inconvenience vs. financial/legal/security risk)
2. Is the rule absolute or context-dependent? (Absolute rules -> hooks; nuanced judgment -> prompts)
3. Can you express the rule in code? (If yes and consequence is high -> hook)
