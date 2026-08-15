---
title: "The Install Constraint, and the Architecture It Forces"
tags: [harness, constraints, architecture, adoption, approval]
status: draft
created: 2026-08-15
---

# The Install Constraint, and the Architecture It Forces

New information, 2026-08-15: the operator **cannot install DeepSeek Harness on
the corporate laptop** — unapproved software is not installable.

Everyone will read this as "so we don't use dsh," which was already the
recommendation. That reading is too small. The constraint is more general than
the example that surfaced it, it applies to things the current design assumes,
and it changes which architecture is correct rather than merely which dependency
is allowed.

---

## 1. The constraint applies to us, not just to dsh

If unapproved software cannot be installed, then the rule does not stop at
DeepSeek Harness. It reaches:

- `enterprise-harness` itself, if it ever ships an endpoint artifact;
- any new CLI, daemon, desktop app, VS Code extension or language runtime;
- `npx anything` — fetching and executing a package *is* installing software,
  and the fact that it leaves no installer entry is a detail of mechanism, not
  of policy;
- and, unless it has already been approved, **EIL**.

### Correcting my own merged document

`ADOPTION_AND_DECOMPOSITION.md` §3 describes rung 0 as deployable now, on the
grounds that it requires "zero new installs beyond what developers already run,"
reasoning that a local Node process is free because Copilot CLI and Amp CLI run
today.

**That reasoning is wrong and I am withdrawing it.** Copilot CLI and Amp CLI are
not present because Node processes are permitted; they are present because those
specific products went through vendor assessment and were approved. Approval
attaches to artifacts, not to runtimes. A new internally-built Node package
inherits nothing from the fact that Node exists on the machine.

Rung 0 is therefore not "deployable now." It is deployable exactly when EIL is
approved, and not one day earlier — which makes the question in §5 the most
valuable one in this document.

---

## 2. The design axis flips

Until now, the drafts have optimised for capability, then security, then scale.
Under this constraint the binding variable is different:

> **Approval surface** — the number of distinct artifacts that require endpoint
> software approval before anyone can use the system.

Every architectural choice should now be scored on it first. A design that is
slightly worse technically and requires one fewer approved artifact is the
better design, because approval has a lead time measured in months and a
non-trivial failure rate, while engineering elegance has neither.

This gives a design invariant that deserves the same weight as "a pack is a
filter, never a grant":

> **The zero-endpoint-install invariant.** No element of this design may require
> a developer to install software that is not already approved. Anything that
> cannot meet that bar ships server-side, rides inside an already-approved
> artifact, or does not ship.

---

## 3. What this does to the three products

The P1/P2/P3 decomposition survives, but the *shape* of each changes, and one of
them changes owner.

### P1 — the pack plane ships **inside EIL**, not beside it

EIL already exposes a concrete extension seam. `ts/tools.ts` holds a flat
`REGISTRY` of six tool specs and a single `callTool()` entry point, and the
README is explicit about what that choke point already carries:

> *"`callTool()` is the single choke point — env gating, argument validation,
> ACL viewer, and audit logging all live there, so you inherit them."*

Pack resolution is read-only. It is search with a declared scope. Adding
`pack_list`, `pack_resolve` and `pack_search` to that registry costs **zero
incremental approval surface**, inherits the ACL viewer and the audit trail for
free, and requires no new process, port, credential or installer.

This promotes what Sonnet framed as the failure branch — *"if every pack is
knowledge-only, fold the concept into EIL and retire this repo"* — into the
**primary design for P1**. Not as a retreat, but because the constraint makes it
strictly better on the axis that now dominates.

The obvious objection is that this pollutes EIL's read-safe trust story. It does
not. What would pollute it is **tool packs**, which invoke things and have side
effects — and those were already deferred to P2 behind an evidence gate. A
read-only scoped search is the same kind of thing EIL already does.

`enterprise-harness` then remains what it is today and should stay: **the repo
where the contracts, decisions and governance model live.** That is real work and
it does not need to be a binary.

### P2 — governed execution is server-side, and that is an easier door

Approvals, sandboxes, mutation tiers and typed plans genuinely do not belong in
EIL. But they also do not belong on the laptop. They ship as a service reached
over HTTPS through the approved egress proxy.

This matters more than it looks: **server-side software follows a different
approval path** — infrastructure change management — which in most enterprises
is a route that already exists and is regularly exercised, whereas endpoint
software approval is a slower, more adversarial queue. Moving P2 off the
endpoint is not a compromise; it is choosing the door that opens.

It is also where per-user identity has to live anyway, so the two constraints
point the same way.

### P3 — measurement was already server-side

The observability ledger and metering gateway are services. No endpoint
component, no change. The shadow arm runs wherever EIL runs.

---

## 4. Two architectures, and one question decides which

Everything above assumes EIL runs on the corporate laptop. If it does not and
cannot, the architecture is different — not worse, but different, and the
dependency ordering inverts.

### Architecture A — EIL is approved and local

```text
Corp laptop  (nothing new installed)
└── Approved agent CLI (Copilot CLI / Amp)
      └── mounts MCP servers
            └── EIL — Node, in-process PGlite, no server, no admin
                  ├── search_docs / get_doc / expand / search_code   (exists)
                  └── pack_list / pack_resolve / pack_search         (P1, added here)

Pack manifests and locks = data in a git repo, not software

Server side  (change-managed, never endpoint-installed)
├── Observability ledger + metering gateway          (exists)
├── Shared EIL + per-user OIDC                        (EIL roadmap dependency)
└── Governed execution service                        (P2, only after evidence)
```

Endpoint approval surface: **zero new artifacts.** EIL was plainly designed for
this laptop — *"no server and no admin rights"*, *"no Docker, no admin"*,
*"zero-install PGlite backend"*, and a documented mode for when Postgres itself
cannot be installed. That is not a coincidence; it is the same constraint
already having shaped a design once.

### Architecture B — nothing is installable, everything is remote

```text
Corp laptop  (nothing installed, ever)
└── Approved agent CLI
      └── mounts a REMOTE MCP server over HTTPS through the corporate proxy
            └── EIL + pack plane, hosted server-side, per-user OIDC
```

Architecture B is viable and in some ways cleaner. But it has a hard
precondition: **remote MCP with per-user identity is the only path**, and EIL's
README lists exactly that as unbuilt —

> *"In progress: Per-user tokens + HTTP MCP transport (phase 2 — the kube
> rollout gate)"*

So under B, the dependency I previously flagged as a *later-phase* gate becomes
the **immediate critical path**, and nothing at all can be piloted until it
lands. Under A, the pilot can start now and that dependency stays where it is —
the gate between per-developer and team-shared use.

That is a large difference in sequencing, and it turns on a one-sentence answer.

---

## 5. The questions that are now worth more than more architecture

Three more design documents will not resolve this. These will:

1. **Is EIL running on the corporate laptop today?** If yes, Architecture A is
   live and the falsification trial can start immediately. If no, we are in B
   and the EIL HTTP/identity milestone is the whole critical path.
2. **What is the approval path and lead time for internally-built software** —
   both endpoint and server-side? This belongs on the critical path *now*, in
   parallel with design, because it is the longest pole and nothing about it
   gets shorter by being discovered later.
3. **Was EIL itself ever approved, or does it run somewhere unconstrained?**
   This is the uncomfortable one, and §6 is about why.

---

## 6. The risk this constraint reveals: building where you cannot deploy

The operator can evidently run EIL, the observability layer and three agents
somewhere. The corporate laptop refuses new software. If those are different
machines, then the entire programme has been designed and validated in an
environment that its deployment target does not resemble.

That is a common and expensive failure. It does not show up as a bug; it shows
up at the end, as a finished system that cannot land — and it is precisely the
failure mode the observability design was already alert to when it wrote *"no
arbitrary software installs"* as **the binding constraint** and ruled out Kafka,
Redis and dedicated vector databases on those grounds.

The mitigation is cheap and should happen before more design: establish what can
actually run on the target machine today, in writing, and treat that inventory
as a design input of the same standing as the threat model. Every subsequent
decision then has a testable answer to "can this be deployed."

---

## 7. What is left of DeepSeek Harness, and it is not nothing

Reading a repository costs no approval. Vendoring code from it creates an
artifact; the MIT licence permits that, but the licence was never the binding
constraint. So: take the ideas, ship none of the code.

Worth taking, from its architecture documents:

- **Capability seams as three explicit roles** — service definition, provider,
  consumer — with the rule that one role alone is not a seam. This is a
  discipline for the contracts in this repo, and contracts are the deliverable.
- **"Model-visible means logged."** Anything reaching a model request must be
  reconstructable from the durable log, asserted as a runtime invariant. This is
  directly applicable to pack resolution: a pack that shaped a request must be
  reconstructable from the evidence, which is the same requirement as
  reproducible runs arriving from a different direction.
- **Interception as typed waterfalls** before request, tool call and turn, rather
  than ad-hoc hooks — the shape P2's policy admission should take when it comes.

What to explicitly reject, which `REFERENCE_ASSESSMENT.md` already argues and
this constraint now makes moot anyway: the plugin tree as a *security* boundary,
and local profile patching as a configuration model.

---

## 8. The constraint is good news, and it should be read that way

It removes options cleanly and early: no fork, no new CLI, no daemon, no Docker,
no desktop application, no second datastore, no endpoint agent of our own. What
survives is smaller, cheaper to build, faster to approve and more likely to
ship.

It also makes the kill-gate experiment cheaper rather than harder. Under
Architecture A the falsification trial needs no new software at all — pack
manifests are data files in a git repo, the scope is an EIL query parameter, and
the shadow arm is a second deterministic query. The most important experiment in
the programme is runnable, today, with nothing installed.

A constraint that deletes this many options this early is worth more than
another month of design.
