# Task 1.2: Orchestrate multi-agent systems with coordinator-subagent patterns

## Knowledge of:

### ◇ Hub-and-spoke architecture where a coordinator agent manages all inter-subagent communication, error handling, and information routing

The coordinator is the single point of control. Subagents never talk to each other directly -- all communication flows through the coordinator. This gives you:
- Centralized error handling
- Consistent information routing
- Full observability of the system's behavior
- A single place to add logging, rate limiting, or policy enforcement

### ◇ How subagents operate with isolated context -- they do not inherit the coordinator's conversation history automatically

Each subagent starts with a fresh context. If the coordinator has been through 20 turns of conversation, a newly spawned subagent knows nothing about that unless you explicitly pass relevant information in its prompt.

### ◇ The role of the coordinator in task decomposition, delegation, result aggregation, and deciding which subagents to invoke based on query complexity

The coordinator's responsibilities:
1. **Decompose** the task into subtasks
2. **Delegate** each subtask to the appropriate subagent
3. **Aggregate** results from subagents
4. **Evaluate** whether the combined output is complete
5. **Iterate** if gaps are found

Simple queries may only need one subagent; complex ones may need several in sequence or parallel.

### ◇ Risks of overly narrow task decomposition by the coordinator, leading to incomplete coverage of broad research topics

Example from the exam: a coordinator decomposed "impact of AI on creative industries" into only visual arts subtasks (digital art, graphic design, photography), completely missing music, writing, and film. The subagents did their jobs correctly -- the problem was the coordinator's decomposition.

Prevention strategies:
- Prompt the coordinator to enumerate broad categories before decomposing
- Include explicit instructions to check for coverage gaps
- Use iterative refinement loops

## Skills in:

### ◆ Designing coordinator agents that analyze query requirements and dynamically select which subagents to invoke rather than always routing through the full pipeline

Not every query needs every subagent. The coordinator should assess complexity and route accordingly. A simple factual lookup doesn't need a full research pipeline with search, analysis, and synthesis agents.

### ◆ Partitioning research scope across subagents to minimize duplication (e.g., assigning distinct subtopics or source types to each agent)

Overlap = wasted tokens and potentially contradictory outputs. Assign clear, non-overlapping scopes:
- By subtopic: "Agent A covers music, Agent B covers visual arts, Agent C covers writing"
- By source type: "Agent A searches academic papers, Agent B searches industry reports"

### ◆ Implementing iterative refinement loops where the coordinator evaluates synthesis output for gaps, re-delegates to search and analysis subagents with targeted queries, and re-invokes synthesis until coverage is sufficient

```
Loop:
  1. Coordinator delegates to subagents
  2. Coordinator receives results
  3. Coordinator evaluates: Are there gaps?
  4. If yes: re-delegate with targeted queries to fill gaps
  5. If no: synthesize final output
```

The coordinator doesn't just delegate once -- it checks if the result is complete and iterates if needed.

### ◆ Routing all subagent communication through the coordinator for observability, consistent error handling, and controlled information flow

Never let subagents call each other directly. The coordinator is the single control plane. Benefits:
- You can log every message for debugging
- You can apply consistent error handling policies
- You can throttle or redirect traffic as needed
