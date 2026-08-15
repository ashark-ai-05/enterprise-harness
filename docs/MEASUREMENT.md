---
title: "Measuring Whether Packs Work"
tags: [harness, measurement, evaluation, observability]
status: draft
created: 2026-08-15
---

# Measuring Whether Packs Work

The central product claim is that giving an agent a curated slice of the org
beats giving it the whole corpus. That claim is **not obviously true**, it is
cheap to test here for a specific technical reason, and one part of the test
expires if we defer it.

This document is about how we find out. It is deliberately short on dashboards
and long on the one decision that has a deadline.

---

## 1. The decision that expires

`main`'s `DELIVERY_PLAN.md` reaches measurement at Phase 4. Sonnet's kill
condition — *"if Stage-0 usage shows scoped packs don't beat mounting the whole
EIL corpus, stop before building a registry"* — is exactly the right test, but
nothing in either plan creates the comparison it needs.

**That comparison cannot be reconstructed later.** Once packs are the paved
road, unscoped runs stop happening. There is no archived control group to dig
out of the ledger, because it was never generated.

This is the same argument the observability design already accepted and locked
for memory: a permanent memory-off holdout, *"which cannot be retrofitted,
because once memory is fully deployed there is no uncontaminated control group
left."* Packs have the identical structure and a nastier failure mode.

Memory's risk is that it costs more than it saves — a *cost* error, visible in
aggregate spend. A pack's characteristic risk is that it narrows the corpus
until the answer falls **outside** it, and the agent then produces a confident,
fluent, correctly-cited, wrong answer. Every citation checks out. Nothing in the
transcript looks wrong. It is invisible without a control.

> **Recommendation: the control instrumentation ships in Phase 1, with the
> walking skeleton. It is not a Phase 4 concern and it is not deferrable.**

This inverts the natural build order, and that is the point. It is also the
cheapest thing on the Phase 1 list, for the reason in §2.

---

## 2. Why the control is nearly free here

EIL's retrieval path has a property most agent systems lack:

> *"Deterministic. Four lexical arms … plus a vector arm … fused with
> reciprocal-rank fusion. Same query, same corpus, same order. **No LLM in the
> retrieval path.**"* — `enterprise-intelligence-layer/README.md`

A counterfactual retrieval therefore costs **a Postgres query, not tokens**. In
a system where the retriever is itself a model call, running both arms doubles
inference spend and nobody does it. Here it is a rounding error against the
observability layer's own <2–3%-of-AI-spend budget.

That makes a design possible that would otherwise be unaffordable.

### Tier 1 — the shadow arm (retrieval quality, always on, sampled)

For a sampled fraction of pack-scoped queries, also run the **unscoped** query
against the caller's full authorised corpus. Do not show it to anyone. Do not
put it in the model's context. Record only which documents each arm returned,
and later join against which documents the run actually **cited in an accepted
outcome**.

That join answers the question directly:

- **Pack precision gain** — did scoping remove documents that were never going
  to be cited? (The claimed benefit.)
- **Pack recall loss** — did scoping remove documents the unscoped arm found
  *and the run would have cited*? (The claimed benefit's hidden cost, and the
  number that kills the product if it is large.)

Recall loss is the metric that matters and the one nobody measures, because
measuring it requires exactly this counterfactual. A pack with 30% precision
gain and 5% recall loss is a good trade. A pack with 30% precision gain and 30%
recall loss is a machine for producing confident errors, and it will report
beautifully on every other metric.

This is a **within-request** control. It degrades nobody's experience, needs no
user cohort, and can run from day one at whatever sample rate the budget allows.

### Tier 2 — the run-level holdout (outcomes, small, permanent)

The shadow arm measures retrieval. It cannot measure whether the *run* went
better, because only one arm actually ran. Outcome questions — time to a
reviewable artifact, first-pass acceptance, rework, turns to completion — need a
real control.

So: a permanent ~5% pack-off holdout at the level of **runs**, not people.
Sampled per run, disclosed in the product, aggregated at team level only.

Sampling runs rather than users matters. A permanent user-level holdout means
some named colleagues get a deliberately worse tool for a year, which is
politically fatal and, in a system that already promises never to measure
individuals, ethically inconsistent. Run-level sampling spreads the cost thinly
and keeps the cohort anonymous.

---

## 3. What to measure, and what to refuse to measure

Everything below is a join over facts the observability ledger already stores.
No pack-specific pipeline is needed — only that every `pack_*` call emits a
canonical event with the pack id, version, lock digest, and `partial` flag.

**Leading indicators — readable in weeks.** These carry the programme.

| Metric | Definition | Why |
|---|---|---|
| Pack recall loss | Documents the unscoped arm surfaced and the run would have cited, that the pack excluded | The failure mode nothing else detects |
| Pack hit rate | Fraction of loaded packs from which at least one document was cited | Measures whether a pack was *needed*, not just present. A low hit rate means the pack is decoration |
| Citation-weighted rot | See `PACK_RESOLUTION.md` §5 | Distinguishes decay that matters from decay that does not |
| Partial-view abstention rate | Fraction of `partial` views that produced an abstention rather than a negative claim | Directly tests the §3 invariant in `PACK_RESOLUTION.md` |
| Tokens to first cited artifact | Pack-on vs pack-off | The economic claim, stated so it can fail |

**Lagging indicators — months.** Real, but do not promise them early.
First-pass acceptance, 30/90-day survival of pack-informed changes, change-
failure rate. The observability design's warning applies unchanged: promising an
executive a change-failure-rate readout in month one is how programmes like this
lose credibility.

**Refused outright**, and worth writing into the charter rather than assuming:

- Individual-level anything. Team aggregation only, in writing, as already
  committed in the observability design.
- Pack counts as a success metric. "247 packs published" measures activity, and
  optimising it produces sprawl. Hit rate and rot are the health metrics; volume
  is not one.
- Self-reported model confidence. Not evidence.

---

## 4. Honest thresholds, set before the data arrives

Sonnet's kill conditions are right in kind but not yet numeric, and a threshold
chosen after seeing the results is not a threshold. Proposed, to be argued with
before Phase 1 rather than after:

- **Stop before building a registry** if, over the Phase 1 window, pack-scoped
  retrieval does not beat unscoped retrieval on citation precision by a margin
  exceeding the measurement's own error bar — with the margin and the error bar
  both stated. "Slightly better" is not a product.
- **Redesign the pack model** if recall loss exceeds precision gain on any
  workflow family. That means scoping is subtracting more signal than noise, and
  no registry, UI or governance layer will fix it.
- **Fold packs into EIL and retire this repo** if pack hit rate stays low —
  agents loading packs they never cite means the pack was never the binding
  constraint, and a saved named query inside EIL does the same job for a
  fraction of the cost. This is Sonnet's §1 argument, made falsifiable.

### On error bars

One piece of local history is worth repeating, because it is the reason this
section exists. In this same programme (work log, 2026-08-13 — worth confirming
against the EIL eval PRs before it is quoted anywhere external), a **+53% MRR
improvement was reported and then retracted** after five validation bugs were
found in the evaluation framework itself: circular subject truth, ACL leaks in
the fixtures, and duplicate queries. The number was wrong in the flattering
direction, which is the direction they usually are. EIL's own history carries
the habit as well — there is a commit whose subject is literally *"retract
unmeasured claim"*.

A pack evaluation set built from the same corpus the pack selects will
reproduce that failure exactly. The eval fixtures must be authored
independently of the pack's selectors, and per-family metrics must be reported
rather than a single headline number, for the same reason they were introduced
there.

If the headline metric moves and no one can say which query family moved it, it
did not move.
