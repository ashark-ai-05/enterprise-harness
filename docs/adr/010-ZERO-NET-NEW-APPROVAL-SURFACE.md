# ADR-010: Zero Net-New Approval Surface

Status: proposed

## Context

The product owner reported (2026-08-15) that `deepseek-harness` could not be installed on their corp laptop under existing unapproved-software policy. ADR-006 already ruled out depending on dsh's runtime for architectural reasons; this is the same failure mode arriving as a fact rather than a risk, and it applies to more than dsh. Whatever `enterprise-harness` becomes is *also* new software someone has to be permitted to run. If its own distribution shape resembles what just got blocked, it fails the identical review for the identical reason — the harness would ship design-perfect and be unable to run on the machine of the person it was built for.

`npx @deepseek-ai/dsh web` has two properties that make it exactly the shape corp software controls target: it fetches and executes code from a registry at invocation time rather than from a reviewed, pinned artifact, and it starts a persistent local web server. Both are common triggers for endpoint-protection and software-asset-management policies independent of who published the package.

EIL does not have this shape. It is cloned source, installed via `pnpm install` against a committed lockfile, and run as `pnpm eil serve` — a subprocess an already-approved agent CLI spawns over stdio, not a standalone listening service. That is presumably *why* EIL has been runnable in this org's agent workflow so far, even though nobody has stated it as a deliberate policy choice until now.

## Decision

`enterprise-harness` adopts EIL's installation shape as a hard constraint, not a style preference:

1. **No fetch-and-run at invocation.** Install from a committed, reviewed lockfile (`pnpm install`), never `npx <pkg>@latest` or an equivalent that pulls unreviewed code from a registry at runtime.
2. **No persistent listening service at Stage 0/1.** The harness is a CLI and an MCP stdio subprocess, spawned by an already-approved agent CLI — the same shape as EIL today. This is now a second, independent reason for the no-web-UI decision already implied by the "not a day-to-day agent" product framing (README, "Product surfaces" notwithstanding — a *console* is a phase-4+ concern for admins, not a Stage-0/1 requirement, and must not be a locally-running daemon when it does arrive).
3. **No new runtime dependency outside what EIL's or Observability's existing lockfiles already carry**, unless a specific need is stated and submitted for review on its own. Every new package is treated as its own approval request, tracked, not bundled invisibly into a larger install.
4. **Pre-clear the installation shape with IT/security in Phase 0**, explicitly, as its own gate — not folded into the general "validate approved Node, Postgres, sandbox, registry choices" line in `DELIVERY_PLAN.md`. That line reads as a technical-fit check; this is a procurement action with a known failure precedent (dsh, this week) and deserves to be tracked as a named risk with an owner, not a checklist item that can quietly slip.

## The open question this doesn't answer

Everything above assumes Node, pnpm, and EIL's own installation are *already* cleared for the product owner's actual corp laptop — the same machine that just rejected dsh. That's plausible (EIL is in active use) but not yet confirmed in anything this repo has written down; it may only be proven for the agents' build/dev environment, which is not guaranteed to be the same trust boundary as the target laptop. If EIL itself turns out to be running under a temporary exception, an unmanaged device, or a different policy tier than the one that blocked dsh, this ADR's whole premise — "match EIL's shape and you clear review" — needs re-checking before Phase 0 relies on it.

**This is the single fact most worth getting a direct answer to before further design work assumes it:** is EIL, running exactly as it runs today, confirmed-approved on the same corp laptop that just rejected `npx @deepseek-ai/dsh web`?

## Consequences

If confirmed, the harness's packaging question mostly resolves itself: ship it as more pnpm-installed TypeScript that composes with EIL's already-approved toolchain, not as a separately-branded product with its own install ceremony — a distribution decision, not a repository-ownership one; the three-plane architecture and separate-repo decision (ADR-001) are unaffected. If not confirmed, the harness may need its own procurement track before Phase 0's technical validation is worth doing at all, since a technically-correct design that can't be run is not a smaller problem than an incorrect one.
