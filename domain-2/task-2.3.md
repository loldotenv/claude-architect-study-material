# Task 2.3: Distribute tools appropriately across agents and configure tool choice

## Knowledge of:

### ◇ The principle that giving an agent access to too many tools (e.g., 18 instead of 4-5) degrades tool selection reliability by increasing decision complexity

More tools = more confusion. Keep each agent's toolset focused on its role. 4-5 tools per agent is a good target.

### ◇ Why agents with tools outside their specialization tend to misuse them (e.g., a synthesis agent attempting web searches)

Tools act as affordances — when an agent *can* do something, it will be tempted to do it, even when that is not its job. This is not a bug in the model; it's a predictable consequence of how tool selection works. The model sees available tools and factors them into its plan.

**Multi-agent research scenario — what goes wrong:**

A research system has three agents: a **Search Agent** (finds sources), a **Synthesis Agent** (combines findings into a report), and a **Coordinator** (routes tasks).

If the Synthesis Agent is given `web_search` in addition to its own tools:

1. It receives partial findings from the Search Agent with a gap (e.g., missing a revenue figure)
2. Instead of flagging the gap and requesting the Coordinator to assign another search, it calls `web_search` itself
3. It finds a different source with a conflicting number, and now must reconcile — which is the Search Agent's job
4. The Synthesis Agent's output mixes verified and unverified sources, undermining the pipeline's quality controls
5. The Coordinator has no visibility into this side search, so it cannot deduplicate or audit

**The fix:** The Synthesis Agent should have only `compile_report`, `flag_gap`, and optionally a scoped `verify_fact` for simple spot-checks. When it encounters a gap, it calls `flag_gap` which routes back to the Coordinator, preserving the division of labor.

### ◇ Scoped tool access: giving agents only the tools needed for their role, with limited cross-role tools for specific high-frequency needs

The principle of least privilege applied to tools. Exam Question 9 demonstrates this: give the synthesis agent a scoped `verify_fact` tool for simple lookups (85% of cases), while complex verifications still route through the coordinator.

### ◇ `tool_choice` configuration options: `"auto"`, `"any"`, `"none"`, and forced tool selection (`{"type": "tool", "name": "..."}`)

| `tool_choice` | Behavior |
|---|---|
| `"auto"` | Claude decides whether to call a tool (default when tools are provided) |
| `"any"` | Claude must call a tool (any tool); no text-only response allowed |
| `"none"` | Claude cannot use any tools (default when no tools provided) |
| `{"type": "tool", "name": "..."}` | Claude must call the specific named tool |

Note: when `tool_choice` is `"any"` or forced, Claude will not emit natural language before the tool call, even if explicitly asked to.

## Skills in:

### ◆ Restricting each subagent's tool set to those relevant to its role, preventing cross-specialization misuse

Apply the principle of least privilege to tool distribution. Each agent should have the minimum toolset to fulfill its role — nothing more.

**Multi-agent research — correct tool distribution:**

| Agent | Role | Tools | Explicitly excluded |
|---|---|---|---|
| Search Agent | Find and retrieve sources | `web_search`, `fetch_url`, `cache_result` | `compile_report` |
| Synthesis Agent | Combine findings into coherent output | `compile_report`, `flag_gap`, `verify_fact` (scoped) | `web_search`, `fetch_url` |
| Coordinator | Route tasks, manage workflow | `dispatch_to_agent`, `check_progress`, `merge_results` | All domain tools |

**Why this matters for the exam:** Question 9 tests whether you understand that a scoped cross-role tool (like `verify_fact` for simple lookups) is better than either giving the synthesis agent full search access or routing every trivial check through the coordinator. The 85/15 rule applies: handle the 85% common case locally with a constrained tool, route the 15% complex cases through the coordinator.

### ◆ Replacing generic tools with constrained alternatives (e.g., replacing `fetch_url` with `load_document` that validates document URLs)

Constrained tools reduce misuse by narrowing what the tool can do.

### ◆ Providing scoped cross-role tools for high-frequency needs (e.g., a `verify_fact` tool for the synthesis agent) while routing complex cases through the coordinator

This is the answer to exam Question 9: give the synthesis agent just enough for the common case (85% simple fact checks), but keep the full pipeline for complex cases (15%).

### ◆ Using `tool_choice` forced selection to ensure a specific tool is called first (e.g., forcing `extract_metadata` before enrichment tools), then processing subsequent steps in follow-up turns

Force the first step, then let the agent handle the rest with `"auto"`.

### ◆ Setting `tool_choice: "any"` to guarantee the model calls a tool rather than returning conversational text

Use when you need structured output and can't accept a text-only response.
