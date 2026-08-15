# ADR-009: Ship Counterfactual Measurement with Packs

Status: proposed

## Decision

The first read-only pack pilot includes a sampled shadow whole-corpus retrieval arm and a small run-level outcome holdout. Measure scoped-pack hit rate, recall loss, retrieval precision, context reduction and accepted outcomes before building the registry or governed execution plane.

Assignments are run-level, never permanent user-level cohorts. Evaluation fixtures are authored independently of pack selectors.

## Consequences

The system preserves the counterfactual before pack usage becomes the paved road. The shadow retrieval arm is inexpensive because EIL retrieval is LLM-free. Additional storage and evaluation discipline are required from the first pilot rather than deferred to analytics work.
