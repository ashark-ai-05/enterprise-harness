---
title: "Opus Draft — What This Branch Adds"
tags: [harness, synthesis, delta]
status: draft
created: 2026-08-15
---

# What this branch adds

This branch is **not** a fourth blueprint. `main` (Codex) and `sonnet-draft`
(Sonnet) already contain independent product, architecture, pack, security and
delivery documents, and they converge on the important things. Restating them a
third time would cost the reader time and produce no new information.

This branch is a **delta**: three documents covering questions that are
unanswered, unasked, or answered wrongly in the existing drafts.

| Document | What it settles |
|---|---|
| [PACK_RESOLUTION.md](PACK_RESOLUTION.md) | The lock/view model — a concrete answer to `PACK_SPEC.md` open question 1, and the reason a pack cannot launder permissions |
| [MEASUREMENT.md](MEASUREMENT.md) | How we find out whether packs actually work. Contains the one decision that expires if we defer it |
| [ADOPTION_AND_DECOMPOSITION.md](ADOPTION_AND_DECOMPOSITION.md) | Distribution as the product strategy, the identity-propagation ceiling, and why this is three products rather than one |

## What I accept from the existing drafts

Stated explicitly so the synthesis owner knows where the disagreements are not.

- **Thin control plane, not a knowledge platform.** EIL owns retrieval; the
  harness owns composition, policy and evidence. (`main` README, ADR-001.)
- **Manifest-first packs.** A pack references EIL resources; it does not copy
  them. (`main` `PACK_SPEC.md`, ADR-002.)
- **A pack is a filter, never a grant.** (Sonnet, thread; `sonnet-draft`
  `ARCHITECTURE.md` §3.) I would fight for this too, and
  [PACK_RESOLUTION.md](PACK_RESOLUTION.md) is largely the mechanism that makes
  it true rather than aspirational.
- **Knowledge packs and tool packs are different trust tiers, not one plugin
  list.** (Sonnet.) `deepseek-harness`'s "everything is a plugin" is a
  composition principle wrongly read as a security boundary — `main`'s
  `REFERENCE_ASSESSMENT.md` says this well and I have nothing to add.
- **Do not fork `deepseek-harness`.** All three of us reached this
  independently, which is about as much confirmation as a design decision gets.
- **Identity propagation is the sharpest risk.** (Sonnet
  `CRITICAL_REVIEW.md` §2.) I extend rather than contest it: see
  [ADOPTION_AND_DECOMPOSITION.md](ADOPTION_AND_DECOMPOSITION.md) §2, where it
  stops being only a security property and becomes the ceiling on how far this
  product can be deployed at all.

## Where I contest the existing drafts

Three places, in descending order of how much I care.

### 1. The delivery sequence put measurement last, and one part of it could not be added later

> **Resolved.** `DELIVERY_PLAN.md` now ships the counterfactual with the Phase 1
> walking skeleton, and the trial has since gained a third arm
> ([MEASUREMENT.md](MEASUREMENT.md)). This section is kept as the record of why,
> not as a live criticism — the tense below is the state at the time of writing.

At the time of writing, `main`'s `DELIVERY_PLAN.md` reached measurement only at
Phase 4. Sonnet's kill
conditions (`CRITICAL_REVIEW.md` §5) are the right instinct — "if scoped packs
don't beat mounting the whole corpus, stop" — but nothing in either plan
establishes the comparison that test requires.

You cannot run that test after rollout. Once every team has a pack for every
workflow, unscoped runs stop occurring and the counterfactual is gone. This is
the same argument the observability design already accepted for the permanent
memory-off holdout, and it applies here with more force, because a pack's
characteristic failure — narrowing the corpus until the answer falls outside it
— produces a *confident, well-cited, wrong* answer that no amount of after-the-
fact log inspection will reveal.

The instrumentation must ship in Phase 1. See
[MEASUREMENT.md](MEASUREMENT.md) §1.

### 2. Freshness is declared in the manifest but nothing says what happens when it is breached

`main` `PACK_SPEC.md` has packs declare whether they prioritise reproducibility
or freshness, and its lifecycle includes "monitor drift". Neither draft defines
query-time behaviour when a pack's sources have moved underneath it.

Silence here defaults to the worst option: serve the stale pinned view and say
nothing. See [PACK_RESOLUTION.md](PACK_RESOLUTION.md) §4.

### 3. All three of us under-read EIL, and it already answers two of our open questions

Every draft agrees ACL is re-evaluated per caller, and none says what the caller
learns from that. I drafted a resolution — and then found EIL had already
shipped one, better than mine, in `ts/coverage.ts`: a `coverage` object with
`complete` / `sources[]` / `requested_absent` / `quarantined_docs`, ACL-scoped
counts, a load-bearing `never_synced → failing → stale → current` ordering, and
answer-level freshness bounds. Its own commit message states the design goal
almost exactly as a pack layer would want it.

The correction is in [PACK_RESOLUTION.md](PACK_RESOLUTION.md) §3–4, and the
lesson generalises past this one section: **the harness's job is smaller than
three drafts assume, because EIL is further along than any of us checked.**
Before Phase 0 specifies anything, someone should read EIL's actual disclosure
and freshness surfaces rather than its README. I did not, on the first pass, and
it cost me two sections.

## On the coordination problem

Sonnet's `CRITICAL_REVIEW.md` §4 is correct that three agents writing adjacent
architecture documents into one empty repo is itself the problem the pack
registry is meant to solve, and that it only pays off with a named synthesis
owner. Codex claimed that role in the thread and has the most complete baseline
on `main`, so my recommendation is that **Codex owns the merge** and this branch
is reviewed as a set of amendments to `main`, not as a competing proposal.

I have deliberately not written a `PRODUCT.md`, `ARCHITECTURE.md` or
`TECH_STACK.md` on this branch. Two exist already and a third would make the
synthesis harder, not better.
