---
title: "Enterprise Harness — Critical Review"
tags: [harness, risk, security, governance, critique]
status: draft
created: 2026-08-15
---

# Critical Review

Argued against the design in `docs/PRODUCT.md`, `docs/USAGE_AND_SCALE.md`,
and `docs/ARCHITECTURE.md` from five angles. Where a concern has a mitigation
already designed in, it's noted; where it doesn't, it's an open question for
`docs/DECISIONS.md`, not something papered over.

## 1. Build-vs-buy: should this exist as a separate repo at all?

The strongest version of "don't build this": EIL could grow a `pack` concept
natively (a saved, named query is a small addition to a system that already
has queries), and "which MCP tools are mounted" is a config concern any
agent CLI already handles. Under this view, `enterprise-harness` is a
coordination cost — a fourth repo (after `eil`, `enterprise-ai-observability`,
and whatever the pack registry's storage lives in) for a feature that could
be two commits into `eil`.

**Why a separate repo anyway:** the trust-tier distinction in
`docs/ARCHITECTURE.md` §4 — tool packs carry side effects and need an
approval workflow EIL has no reason to own. EIL's entire design center is
*read-safe, fail-closed retrieval*; bolting a side-effecting tool-approval
workflow onto it blurs the one property that makes EIL trustworthy today
(§3 below). A separate, thin layer is a real architectural choice, not just
inertia — but it should be revisited if, in practice, every published pack
turns out to be knowledge-only and the tool-pack half never gets used. If
that's true after Stage 0 (`docs/USAGE_AND_SCALE.md`), the honest move is to
fold the knowledge-pack concept into EIL and retire this repo, not keep it
alive for a capability nobody uses.

## 2. Security: the identity-flattening failure mode

`docs/ARCHITECTURE.md` §3 names this but it deserves restating as a failure
scenario, not just a requirement: a harness service sitting between many
users and one EIL instance is structurally the easiest place in this whole
system to accidentally build a confused-deputy. If the harness authenticates
to EIL once (a service account) and enforces "who sees what" purely in its
own pack-scoping logic, then a bug in that logic — not in EIL's own
battle-tested, fail-closed ACL — becomes the entire security boundary. EIL's
README is explicit that an *unstamped* document defaults to owner-only,
fails closed; a harness-side bug fails **open**, because the harness would
be the identity, not the person.

This isn't a hypothetical to design around later — it's the reason
`docs/ARCHITECTURE.md` §3 states identity propagation as a requirement, not
a preference, and it's the single item in this whole design most worth an
adversarial review pass (a real pentest-style pass, not just this document)
before Stage 2 gets anywhere near real credentials.

## 3. Security: tool-pack sprawl and the "everything is a plugin" trap

`deepseek-harness`'s literal "everything is a plugin" framing (Codex,
thread) is elegant for composability and actively dangerous if copied
uncritically into how tool packs get trust. A plugin architecture optimizes
for "easy to add capability"; an enterprise security posture needs "hard to
add capability without review" for exactly the tier-2 tools
(`docs/ARCHITECTURE.md` §4) that can deploy, page, or write. If the harness
ships with a slick `harness pack use <any-mcp-url>` command before the
tier-2 approval workflow exists, the fast path *is* the vulnerability —
people will use whatever's easiest, and "paste an MCP URL and go" will
always be easier than requesting an approval. The design must ship the
friction for tier 2 from day one, not add it after an incident makes the
case retroactively.

## 4. Organizational: three agents building adjacent infrastructure at once

This document is being written concurrently with near-identical documents
from Codex and Opus, all reading the same EIL/observability decisions,
against the same empty repo. That's a live instance of exactly the
coordination problem the harness's pack-registry governance (§1, §2 of
`docs/USAGE_AND_SCALE.md`) is meant to solve for *published packs* — and
it's happening at the level of *architecture decisions themselves*, with no
equivalent registry. Three independent takes converging (or not) on the same
questions is valuable per the original request ("think from different
angles"), but it only pays off if there's an explicit synthesis step with a
named owner and a single resulting decision log — otherwise the org gets
three unreconciled opinions and ships none of them. Concretely: this repo
should not accumulate three parallel permanently-diverging branches; someone
names the merge owner and a deadline, or the human operator does the
synthesis directly.

## 5. What would falsify this design

Stated as explicit kill conditions, not hedged optimism:

- **If Stage 0 usage shows scoped packs don't measurably improve retrieval
  precision or session setup time over "just mount the whole EIL corpus,"**
  the pack abstraction is solving an imagined problem — stop at Stage 0, do
  not build the registry.
- **If, after real usage, every pack anyone publishes is knowledge-only and
  no tool pack with real approval friction ever gets adopted,** fold the
  knowledge-pack concept into EIL directly (§1) and retire this repo rather
  than maintaining a thin shim for a feature EIL could own outright.
- **If the no-arbitrary-installs/corporate-proxy constraint
  (`docs/TECH_STACK.md` §6) turns out to bind hard,** Option C
  (runtime-neutral MCP server, no `dsh` dependency) is not just the
  recommended choice but close to the only viable one — worth confirming
  early since it changes how much of this document is live.
- **If identity propagation (§2) cannot be made to work end-to-end through
  whatever agent CLIs are actually in use** (some may not forward caller
  identity through MCP calls today), Stage 2 is blocked on upstream changes
  to those CLIs, not on anything in this repo — and that dependency should
  be surfaced to the org now, not discovered during a Stage 2 rollout.
