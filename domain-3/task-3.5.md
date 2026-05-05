# Task 3.5: Apply iterative refinement techniques for progressive improvement

### Knowledge of:

- **Concrete input/output examples as the most effective way to communicate expected transformations when prose descriptions are interpreted inconsistently**

  Words are ambiguous. Examples are precise. When Claude interprets your instructions differently than intended, provide 2-3 concrete input->output examples.

- **Test-driven iteration: writing test suites first, then iterating by sharing test failures to guide progressive improvement**

  1. Write tests that define expected behavior
  2. Let Claude implement
  3. Share failures
  4. Iterate until all tests pass

- **The interview pattern: having Claude ask questions to surface considerations the developer may not have anticipated before implementing**

  Tell Claude to ask you questions before implementing. This surfaces edge cases, design considerations, and assumptions you haven't thought about.

- **When to provide all issues in a single message (interacting problems) versus fixing them sequentially (independent problems)**

  | Situation | Approach |
  |---|---|
  | Issues interact (fixing A affects B) | Single message with all issues |
  | Issues are independent | Fix sequentially, one at a time |

### Skills in:

- **Providing 2-3 concrete input/output examples to clarify transformation requirements when natural language descriptions produce inconsistent results**

  "Convert dates to ISO format" is ambiguous. Show examples: `"Jan 5, 2024"` -> `"2024-01-05"`, `"5/1/24"` -> `"2024-01-05"`.

- **Writing test suites covering expected behavior, edge cases, and performance requirements before implementation, then iterating by sharing test failures**

  Tests are a specification language. They remove ambiguity about what "correct" means.

- **Using the interview pattern to surface design considerations (e.g., cache invalidation strategies, failure modes) before implementing solutions in unfamiliar domains**

  "Before implementing this cache layer, interview me about the requirements. Ask about invalidation strategy, TTL, consistency requirements, and failure modes."

- **Providing specific test cases with example input and expected output to fix edge case handling (e.g., null values in migration scripts)**

  Edge case bugs often need a specific failing example to fix, not a prose description.

- **Addressing multiple interacting issues in a single detailed message when fixes interact, versus sequential iteration for independent issues**

  If fixing the date parser also changes how the validator works, describe both together so Claude sees the interaction.
