# Task 5.5: Design human review workflows and confidence calibration

### Knowledge of:

- **The risk that aggregate accuracy metrics (e.g., 97% overall) may mask poor performance on specific document types or fields**

  97% overall accuracy might mean 99.5% on invoices but 80% on handwritten receipts. The aggregate hides the segment-level problem.

- **Stratified random sampling for measuring error rates in high-confidence extractions and detecting novel error patterns**

  Don't just sample randomly. Stratify by document type, field, and confidence level to catch problems that aggregate sampling misses.

- **Field-level confidence scores calibrated using labeled validation sets for routing review attention**

  Have the model output confidence per field, then calibrate those scores against labeled data. "0.95 confidence" should mean ~95% accuracy.

- **The importance of validating accuracy by document type and field segment before automating high-confidence extractions**

  Before removing human review for "high confidence" extractions, verify that confidence is actually calibrated across all document types and fields — not just on average.

### Skills in:

- **Implementing stratified random sampling of high-confidence extractions for ongoing error rate measurement and novel pattern detection**

  Even for high-confidence extractions, keep sampling 5-10% stratified by type/field for ongoing monitoring.

- **Analyzing accuracy by document type and field to verify consistent performance across all segments before reducing human review**

  Check: is accuracy consistent across invoices, receipts, contracts? Across amount fields, date fields, name fields? Only reduce review for segments with proven accuracy.

- **Having models output field-level confidence scores, then calibrating review thresholds using labeled validation sets**

  1. Model outputs confidence per field
  2. Compare against labeled data
  3. Set thresholds: ≥0.95 → auto-accept, 0.80-0.95 → spot-check, <0.80 → human review

- **Routing extractions with low model confidence or ambiguous/contradictory source documents to human review, prioritizing limited reviewer capacity**

  Human reviewers are a scarce resource. Route the hardest cases to them; auto-process the easy ones.
