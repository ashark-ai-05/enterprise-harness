# Enterprise Harness

**Status: design-only. No implementation until explicitly confirmed by the product owner.**

A harness is the thing that turns a model into an agent: it owns the run loop,
decides what context and tools an agent session can see, and emits the
telemetry that lets an org trust what its agents did. Today the org has two of
the three planes an enterprise agent needs, built and shipped:

- **Knowledge plane** — [`eil`](https://github.com/ashark-ai-05/eil): token-free,
  fail-closed-ACL, MCP-served retrieval over Confluence, Jira, code, and notes.
- **Observability plane** — [`enterprise-ai-observability`](https://github.com/ashark-ai-05/enterprise-ai-observability):
  attribution, cost, and outcome tracking for AI coding agents.

It does not have the third: a **runtime/orchestration plane** that binds a
model, a policy, a set of mounted capabilities, and those two planes into one
governed agent session — and that lets an org compose *which* capabilities an
agent gets, per team, per task, without hand-wiring MCP config on every
laptop.

This repo is that design. It proposes **Enterprise Harness**: a thin,
plugin-native runtime that turns EIL corpora and arbitrary MCP tool servers
into composable, named, ACL-scoped **info packs**, loads them into an agent
session on demand, and reports every mount/search/call to the observability
plane by construction — not as an opt-in.

## Read order

1. [`docs/PRODUCT.md`](docs/PRODUCT.md) — what problem this solves, for whom, and why it's not redundant with EIL, deepseek-harness, or Claude Code itself.
2. [`docs/USAGE_AND_SCALE.md`](docs/USAGE_AND_SCALE.md) — how a person, a team, and eventually the whole org actually use it, and what breaks at each stage.
3. [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) — components, the info-pack lifecycle, and how it composes with EIL and observability without duplicating either.
4. [`docs/TECH_STACK.md`](docs/TECH_STACK.md) — language/runtime/plugin-engine choices and why, including the build-vs-adopt call on `deepseek-harness`'s Cordis engine.
5. [`docs/CRITICAL_REVIEW.md`](docs/CRITICAL_REVIEW.md) — the case against building this at all, security and governance risks, and the open questions that need a human decision before any code is written.
6. [`docs/DECISIONS.md`](docs/DECISIONS.md) — ADR-style decision log. Everything here is a proposal pending confirmation, not a locked spec.

## Non-negotiable constraint

**Nothing in this repo authorizes implementation.** It exists to get the
product and architecture right on paper, get it critiqued, and get explicit
sign-off — the same discipline EIL's own `docs/gated-elaboration.html` applies
to requirements artefacts, applied here to the harness itself before a line
of it exists.
