# Task 1.3: Configure subagent invocation, context passing, and spawning

## Knowledge of:

### ◇ The `Agent` tool (formerly `Task`) as the mechanism for spawning subagents, and the requirement that `allowedTools` must include `"Agent"` for a coordinator to invoke subagents

If you forget to include `"Agent"` in the coordinator's `allowedTools`, it simply cannot spawn subagents. This is a configuration requirement, not optional. Note: `"Task"` is still accepted as a legacy alias, but `"Agent"` is the current standard name. Important limitation: subagents cannot spawn other subagents (no infinite nesting).

### ◇ That subagent context must be explicitly provided in the prompt -- subagents do not automatically inherit parent context or share memory between invocations

You must pass all relevant information (findings, instructions, data) directly in the subagent's prompt. There is no implicit context sharing. Each subagent is a fresh conversation.

### ◇ The subagent definition configuration including descriptions, system prompts, and tool restrictions for each subagent type

Each subagent type is defined with:
- **Description**: What it does (used by the coordinator to decide when to invoke it)
- **System prompt**: How it should behave, its role, constraints
- **Tool restrictions** (`allowedTools` / `disallowedTools`): Which tools it can access (principle of least privilege)

```json
{
  "name": "research_agent",
  "description": "Searches the web and academic databases for information",
  "systemPrompt": "You are a research specialist. Find relevant, credible sources...",
  "allowedTools": ["web_search", "academic_search"]
}
```

### ◇ Fork-based session management for exploring divergent approaches from a shared analysis baseline

Session forking (`/fork` command or the SDK `fork` option) creates independent branches from a shared starting point -- useful for comparing different strategies without them interfering with each other. After forking, each branch has its own independent context.

## Skills in:

### ◆ Including complete findings from prior agents directly in the subagent's prompt (e.g., passing web search results and document analysis outputs to the synthesis subagent)

Don't assume the synthesis agent "knows" what the search agent found. Pass the actual findings:

```
Task prompt to synthesis agent:
"Synthesize the following research findings into a coherent report:

[SEARCH AGENT FINDINGS]
- Source 1: ...
- Source 2: ...

[ANALYSIS AGENT FINDINGS]
- Key insight 1: ...
- Key insight 2: ..."
```

### ◆ Using structured data formats to separate content from metadata (source URLs, document names, page numbers) when passing context between agents to preserve attribution

Structure matters for downstream synthesis. Separate the claim from its source so attribution survives:

```json
{
  "finding": "AI adoption in music increased 45% in 2025",
  "source_url": "https://example.com/report",
  "document": "AI in Creative Industries 2025",
  "page": 23
}
```

### ◆ Spawning parallel subagents by emitting multiple `Agent` tool calls in a single coordinator response rather than across separate turns

Parallel spawning = faster execution. If tasks are independent, emit all `Agent` calls in one response. The runtime executes them concurrently.

Sequential (slow):
```
Turn 1: Agent(research_music) -> wait -> result
Turn 2: Agent(research_film) -> wait -> result
```

Parallel (fast):
```
Turn 1: Agent(research_music) + Agent(research_film) -> wait -> both results
```

### ◆ Designing coordinator prompts that specify research goals and quality criteria rather than step-by-step procedural instructions, to enable subagent adaptability

Tell subagents *what* to achieve, not *how* to achieve it:

| Procedural (rigid) | Goal-oriented (adaptive) |
|---|---|
| "Search Google, then search Arxiv, then summarize the top 3 results" | "Find credible sources covering both industry trends and academic research. Prioritize recency and authority." |

Goal-oriented prompts let subagents adapt to what they find during execution.
