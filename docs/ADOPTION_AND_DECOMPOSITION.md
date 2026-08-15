---
title: "Adoption, the Identity Ceiling, and Why This Is Three Products"
tags: [harness, adoption, scale, roadmap, decomposition]
status: draft
created: 2026-08-15
---

# Adoption, the Identity Ceiling, and Why This Is Three Products

`main` and `sonnet-draft` both conclude the harness should be runtime-neutral.
I agree, and I want to argue it from a different direction, because the
architectural argument ("do not fork a dev-preview 45-package monorepo") is the
weaker of the two available arguments and it is the only one currently written
down.

The stronger argument is about distribution. It also produces a concrete,
falsifiable rollout ladder and one dependency on another team's roadmap that
should be raised this week rather than discovered in a Stage-2 rollout.

---

## 1. You cannot win the runtime war, and you do not need to

Developers in this org already have agents. The observability design names
three: **GitHub Copilot CLI, Amp CLI, and internal models-as-a-service
endpoints**. They were chosen by procurement, security review and habit, they
are already through vendor assessment, and they are already in people's muscle
memory.

A new harness that asks a developer to switch runtimes is not competing on
capability. It is competing on switching cost, against tools that already work,
and it will lose regardless of how good it is. This is the ordinary fate of
internal platforms and it is worth naming plainly, because "runtime-neutral" is
being treated in the current drafts as an architectural preference when it is
actually the difference between the product existing and not existing.

The corollary is the useful part:

> **The pack plane's reach equals the set of agent runtimes that can mount an
> MCP server.** That is currently all of them. Any capability that cannot be
> expressed through MCP is a capability only harness-native users get, and the
> harness-native population starts at zero and grows slowly.

This is not a hypothetical integration path. It is precisely how EIL is
consumed today — `claude mcp add eil-knowledge`, an entry in Amp's
`settings.json`, a `.vscode/mcp.json` server — with `callTool()` as the single
choke point where env gating, argument validation, the ACL viewer and audit
logging all live. The pattern is proven in this codebase, by this team, against
these runtimes. Reusing it is the low-risk option, not the ambitious one.

`deepseek-harness` then takes its proper place: **one reference host among
several, adopted as a bundle if and when someone wants a full harness-native
agent** — never the contract. Codex's ADR-003 and `REFERENCE_ASSESSMENT.md`
reach the same place; this is the adoption reason to hold it there even if the
architectural reason softens.

---

## 2. The identity ceiling — the real constraint on how far this deploys

Sonnet's `CRITICAL_REVIEW.md` §2 identifies identity flattening as the sharpest
security risk. It is also, and less obviously, the **binding constraint on
deployment scale**, and the two facts are the same fact.

Here is the awkward part. EIL's security today rests on a property that does not
survive centralisation:

> *"Connectors run on **your** personal credentials — you can only index what
> you could already read."* — `enterprise-intelligence-layer/README.md`
>
> *"Service accounts exist only on kube, after the ACL gate."* —
> `docs/ingestion.md`

A local stdio MCP server runs as the developer, in their own process, against
their own corpus. Identity is *ambient*: it cannot be flattened because there is
only ever one identity present. This is why EIL works today with essentially no
identity machinery.

The instant you share an index across people, that property is gone. The corpus
now contains documents crawled by many principals, and something must decide, per
read, who the caller is. EIL's own README lists the mechanism as **not yet
built**:

> *"In progress: Per-user tokens + HTTP MCP transport (phase 2 — the kube
> rollout gate)"*

So:

> **The harness cannot be deployed above a single-developer footprint until EIL
> ships per-user tokens and HTTP MCP transport. This is a dependency on another
> repository's roadmap, not on anything in this one.**

That belongs in `DELIVERY_PLAN.md`'s decision list today. It is the kind of
dependency that is cheap to sequence around if known in Phase 0 and expensive to
discover in Phase 3, and it is currently absent from both drafts. Sonnet's
falsification #4 circles it; this is the specific, citable version.

---

## 3. The rollout ladder

Three rungs. Each is independently useful, each has a distinct security model,
and the gate between rungs 1 and 2 is §2's dependency. Nothing here requires
choosing the final architecture in advance, which is the point.

### Rung 0 — local, per-developer (deployable, on current evidence)

> **Amended twice on 2026-08-15; read the second amendment.** This rung
> originally read "deployable now" because it needs "zero new installs beyond
> what developers already run." When dsh was blocked I withdrew that, reasoning
> that approval attaches to artifacts rather than runtimes. The operator then
> clarified that **in-house software is permitted and EIL is fine**, which makes
> the original claim roughly correct and my withdrawal an over-correction.
>
> The reasoning was wrong both times, and that is the part worth keeping: rung 0
> is deployable because **first-party code is permitted**, not because a local
> Node process is free, and not because Copilot CLI's presence implies anything.
> See [INSTALL_CONSTRAINT.md](INSTALL_CONSTRAINT.md) §0.

Pack manifests in a git repo. Resolution and serving run in the developer's own
stdio process against their own EIL. Identity ambient and therefore correct; no
central service, no credentials, no infrastructure — but not, as originally
claimed, no approval.

*Gets you:* a real answer to "do packs help", from real work, at essentially no
cost — the Phase 1 measurement in [MEASUREMENT.md](MEASUREMENT.md) runs
entirely at this rung.
*Does not get you:* shared curation, governance, tool packs, or any
organisational effect. Every developer maintains their own corpus, which is
exactly the duplicated-effort problem the product exists to solve.

The tension is worth stating rather than glossing: **the property that makes
rung 0 secure is the same property that stops it scaling.** Rung 0 is a
measurement instrument, not a destination.

### Rung 1 — team-shared, read-only

One shared index per team, packs published to a registry, still read-only, still
no tool packs. This is where ACL-by-ref stops being a design preference and
becomes load-bearing: the corpus now holds documents from many crawlers and
every read must be evaluated against the real caller. It is also where
[PACK_RESOLUTION.md](PACK_RESOLUTION.md) §3's partial-view handling first has
anything to do.

*Gate to enter:* the adversarial ACL pass Sonnet asks for — a genuine
pentest-style attempt to read another principal's documents through a pack, not
a self-review — plus the seeded partial-view fixtures.

### Rung 2 — org-wide, governed, with tool packs

Central HTTP MCP endpoint, per-user OIDC, the full registry, promotion
workflow, mutation tiers and approval friction.

*Gate to enter:* §2's EIL dependency shipped, plus evidence from rungs 0–1 that
packs are used and cited (the hit-rate and recall-loss thresholds in
[MEASUREMENT.md](MEASUREMENT.md) §4).

### Where the no-installs constraint bites

If the observability design's *"no arbitrary software installs"* constraint
still binds — one of the two questions I put to the operator in the thread — the
ladder does not change, but rung 2 stops being optional. Rungs 0 and 1 assume a
developer can run a local Node process, which is already true wherever Copilot
CLI and Amp CLI run today, so they survive. Anything requiring new endpoint
infrastructure does not, and the design should carry no such requirement.

The constraint rules out, as it did for observability: Kafka, Redis, a dedicated
vector database, and any second datastore. The stack answer is therefore already
settled by precedent — **TypeScript on Node 22, pnpm, one Postgres, PGlite for
local and test, MCP as the only external interface** — which matches EIL and the
observability layer exactly, and lets all three share tooling, CI shape and
review habits. `main`'s ADR-005 reaches the same conclusion; I am noting that the
constraint makes it close to forced rather than merely preferable.

---

## 4. This is three products, and the sequence is not the obvious one

`main`'s `DELIVERY_PLAN.md` phases one product. I think there are three
separable ones, each with its own value and its own kill gate, and saying so
protects the useful part if the ambitious part stalls.

| | Product | Value alone | Kill gate |
|---|---|---|---|
| **P1** | **Pack plane** — author, resolve, serve, search packs over MCP | Works with the agents people already use. Deliverable at rung 0 in weeks | Hit rate and recall loss ([MEASUREMENT.md](MEASUREMENT.md) §4) |
| **P2** | **Governed execution** — typed plans, capability admission, approvals, sandboxes, mutation tiers | Only meaningful once packs are used and someone wants to *act* on them | Does any team actually request a tool pack? Sonnet's §1 kill condition |
| **P3** | **Measurement and attribution** — pack events joined to outcomes | Mostly already built in `enterprise-ai-observability`; needs the pack events and the shadow arm | n/a — this is how the others are judged |

Most of P2 is the expensive half of the current design: approval workflows,
sandbox providers, mutation idempotency, four-eyes, kill switches. All of it is
correct and none of it is needed to find out whether the core idea works.

**The sequencing rule, which is the non-obvious part:**

> P1 ships first. **P3's instrumentation ships *with* P1, not after it.** P2
> waits for evidence from P1.

P3-with-P1 is the inversion. Measurement is conventionally last because it is
conventionally expensive; here it is nearly free (§2 of
[MEASUREMENT.md](MEASUREMENT.md)) and one part of it — the control arm — is
destroyed by shipping P1 without it. Building P2 before P1 has evidence is the
standard way these platforms accumulate governance machinery for a capability
nobody adopted.

---

## 5. Scaling in the org, once P1 has earned it

Briefly, because `main`'s `PRODUCT.md` covers the operating model well and I
only want to add three things it does not.

**Packs are owned like repos, not curated like a taxonomy.** Central curation is
a queue, a queue is a bottleneck, and teams route around bottlenecks — Codex's
own risk table says as much. Every pack has a team owner, a review date and a
published health number. The registry ranks; it does not gatekeep. Gatekeeping
applies to tool packs (tier 3+), where the friction is the feature and must
exist from day one rather than being added after an incident.

**Discovery is itself a retrieval problem, and should reuse the retriever.**
"Which pack do I need?" is a search over pack descriptions, owners and usage
signal — so serve it from the same EIL machinery rather than building a second
search stack. Rank by citation-weighted usage, not by recency or alphabet.
(A note from this programme's own history, per the work log of 2026-08-12: EIL's
lexical scorer was found to be breaking score ties alphabetically. That is
exactly the failure that makes a ranked registry look arbitrary to its users,
and it is worth checking for before the registry ships rather than after.)

**Sprawl is handled by decay, not by review boards.** Hit rate and
citation-weighted rot ([PACK_RESOLUTION.md](PACK_RESOLUTION.md) §5) mark packs
stale automatically; unused stale packs deprecate on their review date. No
committee, no cleanup project, no annual audit. The only durable garbage
collection is the kind that runs without anyone volunteering for it.
