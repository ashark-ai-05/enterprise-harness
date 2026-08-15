---
title: "Enterprise Harness — Product Thinking"
tags: [harness, product, eil, mcp, info-packs]
status: draft
created: 2026-08-15
---

# Product Thinking

## 1. The problem, stated precisely

An agent session today assembles its own context ad hoc: whatever MCP servers
happen to be configured in that person's CLI, whatever they thought to paste
in, whatever the model decided to search for. EIL already fixed the *retrieval*
half of this — one deterministic, ACL-safe corpus per install, served over
six MCP tools. But EIL ships **one corpus per install**. It does not have a
concept of "the payments team's pack" versus "the platform team's pack" that
a session can select, combine, or swap. Every agent that mounts EIL gets the
whole indexed org, gated only by document-level ACL at query time.

That's fine at personal-laptop scale (EIL's explicit phase-0/1 target). It
stops being fine the moment more than one team wants to run agents against
*different, curated* slices of org knowledge, or wants to hand an agent a
specific bundle of capabilities for a specific task ("give this session read
access to the PAY Jira project, the payments-service repo, and the on-call
runbook MCP tool — nothing else") rather than the entire org corpus plus
whatever MCP servers happen to be in a config file.

**The harness is the layer that makes "which packs does this session get"
a first-class, governed, auditable decision — instead of an artifact of
whatever's in someone's `~/.mcp.json`.**

## 2. What an "info pack" actually is

The request was "load information packs — confluence, jira, codebases, MCP
tools — search these packs." That phrase conflates two genuinely different
things, and getting the distinction right is the single most important
product decision in this document:

| | **Knowledge pack** | **Tool pack** |
|---|---|---|
| What it is | A named, versioned, ACL-scoped slice of an EIL corpus (e.g. "PAY Confluence + Jira + repo") | A mounted MCP server exposing invokable capabilities (e.g. a deploy tool, a paging tool, a ticketing-write tool) |
| What you do with it | **Search** it — `search_docs`, `search_code`, `get_doc`, `expand` | **Call** it — an action with side effects |
| Trust model | Inherits EIL's fail-closed document ACL | Needs its own allow-list; side effects can't be "gated closed" after the fact the way a search result can |
| Failure mode if wrong | Stale or missing context — annoying, recoverable | An agent takes an action it shouldn't have — potentially irreversible |

A harness that treats these identically — "everything is just a plugin, mount
it and go" — is a security defect wearing a clean abstraction. Knowledge packs
are safe to compose liberally, because EIL's ACL is fail-closed per document.
Tool packs are not, because MCP tool invocation has no equivalent per-call ACL
primitive today — trust is binary at the mount boundary. **The harness's info
packs must be a tagged union, not a flat plugin list**, so that composing five
knowledge packs into a session is a low-stakes operation an individual can do
themselves, while adding a tool pack that can write, deploy, or page someone
is a governed operation with a different approval path.

## 3. Who is this for

Three distinct personas, in order of who benefits first:

1. **Individual contributor running an agent CLI (Claude Code, Codex, Amp,
   Copilot CLI).** Today they either mount raw EIL (whole org corpus, no
   scoping) or hand-configure MCP servers per project. The harness gives them
   `harness pack use pay-team` and the session gets exactly that slice —
   faster retrieval (smaller, relevant corpus beats the whole org for
   precision), and less accidental exposure to docs outside their remit even
   though EIL's ACL would already block reads server-side.
2. **Team lead / platform engineer curating packs for their team.** They
   decide what's in the "PAY team" pack, publish it, version it, deprecate it.
   This is the actual governance surface — see `docs/USAGE_AND_SCALE.md` §3.
3. **Org-level platform/security team.** They need to answer "what could any
   agent in this org read or do right now" — a question EIL alone can't
   answer (it knows what's indexed and who can read which doc, not which
   *sessions* are live with which packs mounted) and observability alone
   can't answer either (it sees what happened, not what's currently possible).
   The harness's pack registry is the only place that question has a clean
   answer, because "what's mounted" is exactly its job.

## 4. What this explicitly is NOT

- **Not another coding-agent CLI.** Claude Code, Codex CLI, Amp, and Copilot
  CLI already are harnesses in the generic sense — they run the loop, decide
  tool calls, talk to a model. Enterprise Harness does not compete with them
  or replace them. It is infrastructure *those* CLIs mount as an MCP layer
  (or a thin SDK), the same way EIL itself is something you `claude mcp add`
  rather than a CLI you interact with directly day to day. If this project
  drifts toward "yet another agent CLI with a REPL," that is scope creep away
  from what's actually missing.
- **Not a fork or replacement of EIL.** EIL's retrieval engine, ACL model,
  ranking, and ingestion pipeline are correct and shipped. The harness
  consumes EIL as a knowledge-pack backend; it does not reimplement search.
- **Not a second observability system.** Every harness-mediated pack mount,
  search, and tool call emits events in the schema
  `enterprise-ai-observability` already defines. The harness is a *producer*
  into that plane, not a parallel one.
- **Not (yet) a web UI.** `deepseek-harness` ships a browser UI as its
  primary surface. EIL deliberately doesn't — it's agent-facing, MCP-native,
  invoked by CLIs and models, not humans clicking around. The harness should
  default to the same posture: its primary consumers are agent CLIs and
  services, not people. A pack-catalog UI is a plausible phase-2+ addition
  for the team-lead persona (§3.2), not a phase-0 requirement.

## 5. Why now, and why this shape

Three things converged in the last week that make this the right time to
design (not build) this:

- EIL shipped enough (`eil serve`, the `REGISTRY`/`callTool` mounting pattern,
  fail-closed ACL, incremental ingestion) that a harness has a real knowledge
  backend to sit on top of, instead of a hypothetical one.
- The observability layer locked its phase-0/1 decisions (Postgres, no
  content capture, team-level-only aggregation, MaaS gateway for
  attribution) — the harness can adopt those decisions rather than
  re-litigating them.
- The org is running three agent identities (Codex, Sonnet, Opus)
  concurrently against the same knowledge and observability infrastructure,
  which is itself the first real multi-tenant pressure test of "does more
  than one agent identity need a different pack mounted at once" — see
  `docs/CRITICAL_REVIEW.md` §4 for why that's also a governance risk, not
  just a vindicating data point.
