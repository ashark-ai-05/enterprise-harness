# ADR-009: Ship Counterfactual Measurement with Packs

Status: proposed

## Decision

The first read-only work uses three retrieval arms: A whole authorised corpus, B equivalent inline selectors without a pack, and C a named/versioned pack. It also includes a small run-level outcome holdout. A → B measures the value of scoping; B → C measures the incremental pack abstraction. Measure recall loss, precision, context reduction and accepted outcomes before building the registry or governed execution plane.

Because a pack's distinctive value is amortized curation, follow the single-developer mechanics trial with Git-distributed manifests across multiple developers and measure use by non-authors. A single-author query cannot prove enterprise pack value.

Assignments are run-level, never permanent user-level cohorts. Evaluation fixtures are authored independently of pack selectors.

## Consequences

The system preserves the counterfactual before pack usage becomes the paved road. The shadow retrieval arm is inexpensive because EIL retrieval is LLM-free. Additional storage and evaluation discipline are required from the first pilot rather than deferred to analytics work.
