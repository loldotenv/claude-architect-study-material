# Task 5.6: Preserve information provenance and handle uncertainty in multi-source synthesis

### Knowledge of:

- **How source attribution is lost during summarization steps when findings are compressed without preserving claim-source mappings**

  "Several studies show X" loses attribution. "Smith (2023) and Jones (2024) found X" preserves it. If the synthesis step compresses without mappings, you can't trace claims back to sources.

- **The importance of structured claim-source mappings that the synthesis agent must preserve and merge when combining findings**

  Each claim should carry its source through the entire pipeline:
  ```
  Claim: "AI adoption grew 40% in 2024"
  Source: McKinsey Report, p.12, published Jan 2025
  ```

- **How to handle conflicting statistics from credible sources: annotating conflicts with source attribution rather than arbitrarily selecting one value**

  Two credible sources say different things? Report both with attribution: "Revenue grew 15% (Source A) or 18% (Source B)." Don't silently pick one.

- **Temporal data: requiring publication/collection dates in structured outputs to prevent temporal differences from being misinterpreted as contradictions**

  "Market size is $10B" (2022 report) and "$12B" (2024 report) aren't contradictions — they're different time periods. Include dates to prevent misinterpretation.

### Skills in:

- **Requiring subagents to output structured claim-source mappings (source URLs, document names, relevant excerpts) that downstream agents preserve through synthesis**

  Subagent output format:
  ```json
  {
    "claim": "AI adoption grew 40%",
    "source_url": "https://...",
    "document": "McKinsey AI Report 2025",
    "excerpt": "Our survey found a 40% increase...",
    "publication_date": "2025-01-15"
  }
  ```

- **Structuring reports with explicit sections distinguishing well-established findings from contested ones, preserving original source characterizations and methodological context**

  Report structure:
  - **Established findings** (multiple sources agree)
  - **Contested findings** (sources disagree — show both with attribution)
  - **Limited coverage** (only one source, or source access failed)

- **Completing document analysis with conflicting values included and explicitly annotated, letting the coordinator decide how to reconcile before passing to synthesis**

  Pass conflicts up — don't resolve them prematurely. Let the coordinator or synthesis agent handle reconciliation with full context.

- **Requiring subagents to include publication or data collection dates in structured outputs to enable correct temporal interpretation**

  Dates prevent false contradictions and enable the synthesis agent to present a timeline view.

- **Rendering different content types appropriately in synthesis outputs — financial data as tables, news as prose, technical findings as structured lists — rather than converting everything to a uniform format**

  Match the output format to the content type. A table of financial figures communicates better than a paragraph describing the same numbers.
