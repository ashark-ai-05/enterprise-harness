---
title: "Enterprise Harness — Usage and Org Scaling"
tags: [harness, scaling, rollout, governance]
status: draft
created: 2026-08-15
---

# Usage and Org Scaling

Three stages, matching the phase structure EIL and observability already
committed to (`docs/golden-queries.md`-style incremental rollout, "per-user
tokens + HTTP MCP transport" as EIL's own stated phase-2 gate). Each stage
below states what breaks if you try to skip to it early.

## Stage 0 — individual, laptop-local (today's EIL model, unchanged)

One person, one laptop, one EIL install (PGlite, no server, no Docker — EIL's
existing zero-install path). The harness here is a thin CLI wrapper:

```sh
harness pack use pay-team          # mounts a knowledge pack scoped to a saved query/filter over the local EIL corpus
harness pack use deploy-tools      # mounts an MCP tool server, after an explicit trust prompt
harness run -- claude              # launches the agent CLI with exactly those packs' MCP tools in scope
```

At this stage a "pack" is just a **named filter plus a mount list** stored in
a local config file — there is no registry, no server, no multi-user
concern. This is deliberately the same shape as EIL's own bootstrap: prove
the abstraction works for one person before making it a service.

**What this stage validates:** that splitting one EIL corpus into
task-scoped packs actually improves retrieval precision and reduces
irrelevant context, versus just mounting the whole corpus. If it doesn't,
stop — the rest of this document is solving a problem that doesn't exist.
See `docs/DECISIONS.md` D1.

## Stage 1 — team, shared pack catalog

A team publishes packs others can `harness pack use` without knowing the
underlying EIL query syntax. This requires:

- A **pack registry** — versioned, at minimum a Git-tracked manifest
  (pack name → EIL query/scope definition + mount list), not necessarily a
  running service yet. Team leads own their team's packs the way a
  `CODEOWNERS` file works, not through a UI.
- **No change to EIL's trust model.** Ingestion still runs on personal
  credentials ("you can only index what you could already read" — EIL
  README). A published pack is a *view* over an already-ingested corpus, not
  a new ingestion path. This matters: publishing a pack cannot be used to
  grant access to data the consuming user couldn't already query directly
  through EIL's own ACL.
- **Staleness and deprecation policy.** A pack that references a query over
  a corpus that's since been re-scoped or a repo that's been archived needs
  to fail loudly (`harness pack use pay-team` → "pack references 0 live
  documents, last validated 40 days ago") rather than silently return empty
  results. This is the same "detectable tampering" discipline EIL's
  `reqs check` applies to requirements artefacts, applied to packs.

**What breaks if you skip here:** without a registry, every team reinvents
its own pack definitions locally, which is worse than the status quo (no
packs) because now there are N inconsistent, undiscoverable scoping schemes
instead of one obviously-too-broad one.

## Stage 2 — org-wide, server-enforced

This is EIL's own stated "phase 2 — the kube rollout gate": per-user tokens,
HTTP MCP transport, a real service instead of a personal laptop process. The
harness's pack registry becomes a service at this point too, for the same
reason EIL's does — **local enforcement only works when the enforcer and the
enforced are the same trust boundary.** Once packs are org-published and
consumed by many people's sessions, "which packs can this identity mount"
has to be a server-side decision keyed on the caller's real identity, not a
config file on their laptop that they could edit.

Concretely, this stage requires:

- Identity propagation from the agent CLI → harness → EIL, so EIL's
  fail-closed document ACL is evaluated as the *actual calling user*, not a
  shared service account. (This is already implied by EIL's own ACL design;
  the harness must not become the thing that flattens per-user identity into
  one credential when it proxies calls — see `docs/CRITICAL_REVIEW.md` §2.)
- A tool-pack approval workflow distinct from knowledge-pack publishing,
  because tool packs carry side effects (§2 of `docs/PRODUCT.md`). Who can
  publish a tool pack that lets an agent deploy or write to Jira is a
  narrower, higher-scrutiny list than who can publish a knowledge pack.
- Reuse of the observability plane's existing scale decisions rather than
  re-deriving them: Postgres, not a message broker (session/pack-mount
  volume is the same order of magnitude as the agent-session volume
  observability already sized for); team-level aggregation only, **never**
  per-individual usage stats surfaced anywhere the harness's own telemetry
  is visible (`AI_OBSERVABILITY_DECISIONS_OPUS.md` D13 — this constraint
  applies with equal force to "who mounted which pack how often," which is
  exactly the kind of metric that reads as surveillance if exposed
  per-person).

**What breaks if you skip here and try to go org-wide on the Stage-0/1
model:** the laptop-local trust model has no way to prove that a session
claiming to be "Alice, PAY team" actually is Alice — anyone with the config
file can mount any pack. That's an access-control regression versus EIL's
current per-document ACL, not an improvement.

## Cost and adoption shape

Unlike the observability layer (which has to ingest telemetry regardless of
whether anyone acts on it), the harness only costs anything when someone
opts to use scoped packs instead of raw EIL. That makes rollout naturally
incremental and low-risk: EIL keeps working exactly as it does today for
anyone who never touches the harness. The harness is additive scoping on
top, not a replacement anyone has to migrate to. This is the strongest
argument for building it as a separate thin layer rather than folding pack
concepts into EIL itself — see `docs/CRITICAL_REVIEW.md` §5 for the
counter-argument.
