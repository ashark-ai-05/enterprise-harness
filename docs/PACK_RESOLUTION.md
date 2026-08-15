---
title: "Pack Resolution — Locks, Views, Partial Answers and Rot"
tags: [harness, packs, acl, security, freshness]
status: draft
created: 2026-08-15
---

# Pack Resolution

`docs/PACK_SPEC.md` on `main` defines what a pack *is*. This document defines
what happens when one is *used*, which is where the security properties are
actually won or lost.

It answers open questions 1 and 3 from that document concretely, and raises two
it does not ask.

---

## 1. The lock pins identity, not bytes

`PACK_SPEC.md` open question 1:

> Should live scopes pin every resource ID or pin the query plus EIL index
> generation?

**Neither on its own. Pin resource identity and content digest; never pin
content.**

Resolving a pack produces a `pack.lock` — an ordered list of
`(resource_id, content_digest, source_cursor)` triples, plus the query
expressions that produced them and the EIL index generation they were evaluated
against. The lock records *which documents, in which revision*. It records none
of their text.

This is the whole trick, and it buys three properties that otherwise conflict:

**Reproducibility.** A run cites `pack:payments-oncall@4.2.0+lock:sha256:…`.
Replaying it later re-resolves the same resource identities at the same digests,
so "why did the agent conclude that" has a deterministic answer. EIL's retrieval
is already deterministic — same query, same corpus, same order — so pinning the
corpus is the only missing ingredient for reproducible runs.

**Non-laundering.** Because the lock holds no bytes, it is not a capability.
Handing someone a lock hands them a *bibliography*, not a library. Reading
anything still goes through EIL, which enforces its own fail-closed ACL against
the real caller. A pack therefore cannot expose anything the consuming identity
could not already read — the property Sonnet identified as the one worth
fighting for — and it is true *structurally* rather than by careful
implementation.

**Auditability of divergence.** Because the digest is pinned, drift is
computable rather than merely suspected (§4).

The cost is honest and should be stated: a lock is not portable to a
disconnected environment, because resolving it requires EIL. That is the correct
trade, and it is why `main`'s "snapshot pack" — the encrypted, TTL-bound,
content-carrying export — must stay a separate, exceptional profile with a
different name and a different approval path. A snapshot *is* a capability. It
should never be called a pack in a UI.

### Query-shaped scopes

For a live scope like `jira: project = PAY AND type = Incident`, resolution
enumerates the matching resource identities and locks them like any other. It
does **not** store the query as the pin, because a stored query re-evaluated
tomorrow silently changes what a "reproduced" run saw. Storing both is not
belt-and-braces; it is two different answers to one question.

The query expression is retained as *provenance* — how this set was chosen — and
the resolved identities are the pin. When a live pack is re-resolved on a
schedule, each resolution is a new lock with its own digest, and runs cite the
lock they actually used.

---

## 2. A view is a lock intersected with the caller

```text
view(pack, caller) = lock(pack) ∩ readable(caller, at read time)
```

Two people loading `payments-oncall@4.2.0` get different views, and this is
correct rather than a defect to be designed away. The pack is a *statement of
relevance* ("this is the payments on-call world"); the caller's ACL is a
*statement of authority*. Merging them at authoring time — pre-computing a view
per group, say — reintroduces exactly the laundering problem, because the
pre-computed view outlives the authorisation that produced it.

Three consequences worth writing into the schema:

- **No view is cached across principals.** A view may be memoised within one
  run, bounded by that run's lifetime, and must be discarded on revocation
  signal.
- **Pack composition intersects, never unions, policy.** If pack A depends on
  pack B, the resulting view is `view(A) ∩ view(B)` for shared resources and the
  union of *identities* — but every identity is still ACL-checked individually,
  so a union of identities cannot widen authority. `main`'s rule that "overlays
  may narrow but never broaden parent policy" is the same instinct; the lock
  model makes it automatic instead of a check someone must remember to write.
- **Revocation is a read-time property.** `main`'s open question 3 asks the
  maximum acceptable delay between source ACL revocation and EIL enforcement.
  Under this model the harness contributes *zero* delay of its own — it holds no
  content and caches no authority — so the answer reduces entirely to EIL's own
  connector re-sync interval. That is the right place for the question to live,
  and it should be restated as an EIL SLO rather than a harness one.

---

## 3. What a caller learns about the part they cannot see

**Correction to my own first draft.** I wrote this section as an open problem
with a novel resolution. It is not open — **EIL already solved it**, and the
harness's job is to extend the existing mechanism rather than invent a parallel
one. Proposing a second disclosure object would have created exactly the failure
the EIL commit history warns about: *"two definitions of 'stale' is how a
dashboard ends up disagreeing with the tool everyone actually reads."*

What exists today, in `ts/coverage.ts` and `ts/search.ts`: every EIL answer
carries a `coverage` object.

```text
complete          nothing known to be missing
sources[]         per-family state, last success, run and item failures
requested_absent  sources the caller asked for that have no connector here
quarantined_docs  evidence withheld that this viewer could otherwise read
```

Source states rank `never_synced → failing → stale → current`, and the order is
load-bearing: a family aggregates to its *sickest* scope, and `stale` clears
`complete` because "complete is a positive claim, so unknown must not read as
true." Answers also carry `corpus_current_to`, `evidence_as_of_oldest` and
`evidence_as_of_newest`, with the oldest bound null if *any* citation lacks a
timestamp.

The motivating insight is precisely the one a pack layer needs:

> *"the absence of a SOURCE reads as the absence of evidence — so 'nothing
> matched' and 'that system has not answered since Tuesday' arrive in the
> caller's context window wearing the same shape. The first invites a
> conclusion; the second forbids one."*

The leak question is settled there too, in the direction I would have argued
for: `quarantined_docs` is scoped to the viewer's ACL rather than the tenant,
because *"a tenant-wide count would tell every viewer how many documents exist
that they have no right to know about"* — and `last_error` is never returned to
a viewer at all, since source error strings routinely carry hostnames and
occasionally a credential in a query string.

### What packs actually add, and it is one field

`coverage` reports on **sources**. It cannot report the dimension a pack
introduces — that the caller's *view of a pack* is narrower than the pack's
lock — because ACL exclusion is invisible by construction: visibility is stamped
on the document, and an unstamped document is owner-only.

So: **extend `coverage` with one pack-scoped field. Do not build a second
disclosure object.**

```text
pack_view_partial   true when |view(pack, caller)| < |lock(pack)|
```

Boolean only — no count, no ratio, no list. That follows directly from the
`quarantined_docs` precedent, since a *per-pack* count is a narrower and
therefore better side channel than the tenant-wide one already rejected. It
clears `complete` for the same reason `stale` does.

Two consequences, both already implied by the existing design:

1. **It reaches the model, not just an audit log.** The entire point of the
   coverage work is that the distinction arrives *in the caller's context
   window*. A pack-partial view must render there as an explicit statement that
   absence of evidence is not evidence of absence.
2. **Where attribution is safe it comes from `sources[]`,** at container
   granularity, and only for containers the caller can already observe exist in
   the source system. Where it is not safe, the view carries the pack owner's
   **escalation contact** — the answer to "I might be missing something" is a
   named human, not more metadata, and a widening request leaves an audit trail
   that a count would have leaked.

The invariant, phrased so it can be a test rather than a principle:

> **A partial view must never render as a complete answer.** For any query where
> `pack_view_partial` is true and the result set is empty, the output must
> contain an abstention rather than a negative claim.

That belongs in the same red-team suite as EIL's existing 11 ACL scenarios, and
it should be verified the way the coverage work was — by confirming that seeded
mutations actually fail the tests, not merely that the tests pass.

---

## 4. Freshness: EIL measures it, the pack declares, the harness enforces

Same correction, smaller scope. EIL already **bounds and reports** an answer's
freshness — `corpus_current_to` is the *oldest* successful sync across the
tenant's cursors rather than the newest, specifically so one freshly-synced
scope cannot vouch for the stale ones beside it.

What does not exist, and is genuinely the harness's job, is a **policy**. EIL
can tell you the evidence is eleven days old; nothing decides whether an
eleven-day-old on-call runbook may be served. `PACK_SPEC.md` has a pack declare
whether it prioritises reproducibility or freshness, but nothing defines
query-time behaviour when that declaration is breached, and the default of
silence is the worst available option.

The division of labour is clean and worth keeping explicit: **EIL measures, the
pack declares, the harness enforces.** Each source selector carries a freshness
contract:

```yaml
sources:
  - confluence: { space: PAY, query: "label = runbook" }
    freshness: { maxAge: 7d, onBreach: degrade }
```

`onBreach` takes one of three values, and there is no fourth:

| Value | Behaviour when the lock is older than `maxAge` |
|---|---|
| `degrade` | Serve live EIL search over the same selectors instead of the lock. Results are stamped `resolution: live-fallback`. Reproducibility is lost for this query and the result says so. |
| `refuse` | Return no results for that selector, with a typed reason. For packs where a stale answer is worse than no answer — active incidents, current on-call, live config. |
| `serve` | Serve the lock anyway. Legitimate for genuinely immutable corpora — a shipped API version, a closed post-mortem, a regulation as-at a date. Must be justified in the manifest, because it is the option that hides problems. |

`degrade` is the default. `serve` requires a written justification field that
appears in the pack review.

The important part is the stamp, not the policy. A result that came from a live
fallback is a different kind of evidence from a result that came from a pinned
lock, and the run's evidence bundle must distinguish them. Otherwise "we can
reproduce this run" becomes a claim the system cannot actually honour, and
discovering that during an audit is expensive.

---

## 5. Pack rot, weighted by what anyone actually reads

Rot is computable because the lock pins digests. For a pack at time *t*:

- **drifted** — lock entries whose current source digest differs from the pinned one
- **vanished** — lock entries EIL has tombstoned
- **rot** = (drifted + vanished) / |lock|

Raw rot is a weak signal on its own. A 200-resource pack where the 40 stale ones
are appendices nobody has ever cited is healthy; a 200-resource pack where the
single most-cited runbook is stale is dangerous, and both report 20%.

So publish **citation-weighted rot**: weight each lock entry by how often it was
cited in accepted outcomes over the trailing window. This is available for free —
the observability ledger already records citations per run, so the weights are a
join, not new instrumentation.

Proposed handling:

- Rot is published on the pack's registry page as a first-class number, next to
  the owner. Not buried in a dashboard.
- Crossing a citation-weighted threshold moves the pack to `stale` and notifies
  the owner. It does not auto-deprecate; a stale pack that still beats no pack is
  common, and automatic removal would be a self-inflicted outage.
- A pack that stays `stale` past its review date is deprecated automatically.
  This is the garbage-collection story for pack sprawl, and it is usage-driven
  rather than committee-driven, which is the only kind that survives contact with
  an org.

---

## 6. The injection invariant this makes testable

`main`'s `SECURITY_AND_GOVERNANCE.md` has a good indirect-injection checklist,
including "block retrieved content from changing tool permissions". I want to
state the structural version, because a checklist item is a thing someone can
forget and an invariant is a thing CI can enforce.

Confluence pages and Jira comments are **writable by a large fraction of the
org**. Any pack that draws content from them is, by construction, carrying
attacker-influenced text. That is tolerable. What is not tolerable is that same
step also holding authority to act.

> **Tool grants are a property of the signed manifest and a pure function of
> it. No retrieved content, tool output, or model output may participate in
> computing them.**

And the coupling rule that follows, which nothing in the current drafts states:

> **A step whose context includes content from a broadly-writable source may
> not hold a tier-3 or higher capability** (the mutation tiers in
> `SECURITY_AND_GOVERNANCE.md`).

Read the runbook, or file the change — not both in one step. Splitting them
costs a plan step and an approval boundary, and it converts prompt injection
from a privilege-escalation vector into a text-quality problem. Sources carry a
writability classification for exactly this evaluation, and it comes from the
connector, not from the pack author, because pack authors will get it wrong in
the optimistic direction.

---

## 7. A pack scope is a predicate, never a second read path

Sonnet's recommendation, promoted here from an experiment convenience to an
architectural rule, because there is a stronger reason for it than the one that
surfaced it.

> **Pack resolution restricts the existing query. It does not introduce a new
> one.** In SQL terms, `AND d.id = ANY($lock_ids)` composed into the same
> `c.tsv @@ qq.loose` query that already carries `visibleSql()` and the
> `sources` filter — never a fetch of locked document ids on its own path.

Four reasons, in ascending order of how much they matter.

**1. It is the only pattern this codebase uses.** `sources` rides as
`AND ($7::text[] IS NULL OR d.source = ANY($7::text[]))` on the FTS-driven
query today, and arm B's `hierarchy` selector would ride the same way. A pack
scope is the same kind of thing and should look like it.

**2. A direct fetch returns an unranked set.** Pulling every locked id back
gives the agent documents in no relevance order — worse than arms A and B for
an agent, and not comparable to them on retrieval *quality*, which is the metric
the trial actually turns on. This alone disqualifies the direct-fetch shape as a
product, independently of measurement.

**3. It makes the latency comparison sound by construction.** Codex's rule
currently makes reinstating B → C latency conditional on arm C's access path. If
arm C is a predicate, that condition is satisfied by design rather than checked
afterwards, and the metric survives.

**4. EIL has already paid for a second read path once, and the receipt is in
the history.** This is the reason that should settle it.

`visibleSql()` is not a gate the database enforces; it is a clause each read
path has to *remember to compose*, and it appears at five separate call sites in
`ts/search.ts` alone. When one path forgot, the result was exactly what you
would expect:

> *"The code shortcut bypassed A4's validity filter entirely. `search_code` has
> its own read path with hand-written ACL clauses, so `visibleSql` never reached
> it and a retired file stayed citable long after prose search stopped returning
> it. 'It is in the one predicate every arm composes' was true of every arm
> except the one that did not compose it. **A choke point only protects the
> paths that pass through it.**"*

A fetch-by-lock-id arm would be a new read path with its own hand-written
clauses — the same shape, in the same file, as the bug that was already found
and fixed here. Adding one to serve a pack feature would be repeating a mistake
this repository has already documented in its own commit message.

### Why this follows from the lock model rather than merely fitting it

Section 1 calls a lock a bibliography rather than a library. A predicate is what
that metaphor means in SQL: the lock supplies **candidacy**, the FTS arms supply
**relevance**, and `visibleSql()` supplies **authority**. Three separate jobs,
composed into one query.

Fetching by locked id collapses candidacy into the answer — it treats the lock
as a result set rather than a scope, which is precisely the reading of a pack
that §1 rejects for reproducibility and §2 rejects for authorisation. So the
predicate shape is not an implementation preference that happens to suit the
trial. It is what a content-free lock already commits us to, and the trial is
simply the first place the commitment becomes checkable.

### What this rule does *not* prohibit

The invariant is about **candidacy**, and it needs one carve-out stated
explicitly, because the short form — "a direct fetch-by-lock-id read path is
prohibited" — can be read to forbid something EIL's contract depends on.

EIL is **two-phase by design**: *"search returns ids and snippets, the agent
fetches only what it actually needs."* The second phase is `get_doc(id)`, and it
is a fetch by id. It is also already correct: its query is
`FROM documents d WHERE id = $1 AND visibleSql(2, 3, 4, includeSuperseded)` —
it composes the shared clause rather than hand-writing its own, which is exactly
what the `search_code` path failed to do.

So, precisely:

| Shape | Status |
|---|---|
| Pack scope as `AND d.id = ANY(lock_ids)` on the FTS query | **required** |
| `get_doc(id)` on a document surfaced by pack-scoped search | **allowed** — it is phase two, and it already composes `visibleSql()` |
| Returning the lock's documents *in place of* searching them | **prohibited** |
| Any new read path that hand-writes its own ACL clauses | **prohibited** |

The distinction is not fetch-versus-search. It is whether a path **composes the
shared visibility clause** and whether the lock is being used as a **scope** or
as an **answer**. An implementer who reads the short form literally could
conclude that a pack-resolved document may never be fetched, which would break
the two-phase contract every EIL client already depends on.
