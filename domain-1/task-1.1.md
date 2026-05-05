# Task 1.1: Design and implement agentic loops for autonomous task execution

### Knowledge of:

- **The agentic loop lifecycle: sending requests to Claude, inspecting `stop_reason` (`"tool_use"` vs `"end_turn"`), executing requested tools, and returning results for the next iteration**

  The core loop: send message -> check stop_reason -> execute tools -> append results -> repeat.

  ```python
  while True:
      response = client.messages.create(model=model, tools=tools, messages=messages)
      messages.append({"role": "assistant", "content": response.content})

      if response.stop_reason == "end_turn":
          break
      if response.stop_reason == "tool_use":
          results = [execute(b) for b in response.content if b.type == "tool_use"]
          messages.append({"role": "user", "content": results})
  ```

  `stop_reason` is the **only** reliable loop control signal. Other possible values include `"max_tokens"` (hit the limit), `"stop_sequence"` (custom stop sequence encountered), `"pause_turn"` (server tool pausing), and `"refusal"` (request refused).

- **How tool results are appended to conversation history so the model can reason about the next action**

  Each iteration appends: (1) Claude's response, (2) tool results. This growing message array is the agent's session memory. Skip appending -> Claude loses context, may re-call tools or hallucinate results.

- **The distinction between model-driven decision-making and pre-configured decision trees or tool sequences**

  Model-driven: Claude picks which tools to call and in what order. Pre-configured: your code hardcodes the sequence. Agentic = model-driven. Your code just executes and feeds back.

### Skills in:

- **Implementing agentic loop control flow that continues when `stop_reason` is `"tool_use"` and terminates when `stop_reason` is `"end_turn"`**

  Claude can return text *alongside* tool calls -- `stop_reason` will still be `"tool_use"`. Don't be fooled by the presence of text. Always check `stop_reason`, not content.

- **Adding tool results to conversation context between iterations so the model can incorporate new information into its reasoning**

  Each `tool_result` must include the matching `tool_use_id` so Claude connects the result to the right call.

  ```python
  tool_results = []
  for block in response.content:
      if block.type == "tool_use":
          output = execute_tool(block.name, block.input)
          tool_results.append({
              "type": "tool_result",
              "tool_use_id": block.id,
              "content": output
          })
  messages.append({"role": "user", "content": tool_results})
  ```

- **Avoiding anti-patterns: parsing natural language signals to determine loop termination, setting arbitrary iteration caps as the primary stopping mechanism, or checking for assistant text content as a completion indicator**

  | Don't | Why | Do instead |
  |---|---|---|
  | `if "I'm done" in text` | Phrasing is unpredictable | `stop_reason == "end_turn"` |
  | `for i in range(5)` as primary exit | Too rigid for variable-length tasks | `stop_reason` + safety cap as fallback |
  | `if has_text and no tool_calls` | Text can appear with tool calls | `stop_reason == "end_turn"` |

  A safety cap (max iterations) is acceptable as a **fallback**, but never as the primary termination mechanism.
