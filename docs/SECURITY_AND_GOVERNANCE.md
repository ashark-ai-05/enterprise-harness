# Security and Governance

## Threat model

The harness processes untrusted natural language, enterprise content, tool output and plugin code. Threats include prompt injection, confused-deputy access, cross-tenant leakage, excessive tool authority, credential theft, plugin supply-chain compromise, replayed mutations, telemetry exfiltration and misleading self-certified outcomes.

## Non-bypassable control path

Every retrieval, model call and tool execution must pass through a host-owned gateway that binds:

`principal + tenant + purpose + run + step + pack + capability + data class + budget`.

Models never receive raw credentials. Workers receive short-lived, audience-bound tokens scoped to one admitted step. Tool schemas exposed to the model are derived after policy evaluation.

## Authorization

- Enterprise SSO and workload identity; group sync and service principals.
- Source ACLs remain authoritative and are enforced by EIL at search and fetch.
- Harness policy may further restrict access but cannot grant beyond EIL/source authority.
- Separate permissions for pack publication, workflow publication, execution, approval, sensitive reveal and administration.
- Four-eyes approval for production mutations, policy changes and high-risk capability promotion.
- Emergency access is time-bound, reasoned, separately alerted and audited.

## Prompt-injection containment

- Treat all retrieved content and tool output as untrusted data, never instructions.
- Keep system policy and capability admission outside retrieved context.
- Label context by source, trust and classification.
- Require typed tool arguments and validate them after the model call.
- Block retrieved content from changing tool permissions, destination, secrets or approval state.
- Apply egress controls and data-loss prevention at model/tool gateways.
- Test packs with seeded indirect-injection cases before promotion.

## Plugins and supply chain

- Trusted registry, signed artifacts, publisher verification, SBOM and vulnerability scanning.
- No arbitrary install from a public registry in production.
- Deny undeclared network/filesystem/secret access.
- Run I/O plugins out of process with resource limits and read-only roots by default.
- Pin exact digests in published workflows; changes require promotion and canarying.
- Central kill switch and revocation list; inventory all active plugin versions.

## Data handling

- Metadata-first telemetry. Prompt, response, source bodies and tool arguments are off by default.
- Classify before persistence; encrypt payload classes with separate keys and access paths.
- Store references/digests where possible; audited reveal for sensitive bodies.
- Propagate retention, deletion, legal hold and residency to run artifacts and derived data.
- Never capture hidden chain-of-thought. Store observable inputs, actions, outputs, decisions and evidence.
- Do not use the system for individual performance ranking.

## Mutation safety

Capability tiers:

| Tier | Example | Default control |
|---|---|---|
| 0 | search public/low-risk pack | policy, audit |
| 1 | fetch confidential resource | ACL, purpose, audit |
| 2 | write disposable sandbox | isolated workspace, budget |
| 3 | create Jira comment / draft PR | explicit scope, preview, approval by policy |
| 4 | merge, deploy or production action | external control + human approval; generally deferred |

Mutations require idempotency keys, precondition/version checks, dry-run or preview when supported, and a reconciliation path for unknown completion. Compensation is explicit; “rollback” is not assumed possible.

## Audit evidence

Retain signed metadata for identity, policy version and decision, pack/capability digests, plan revisions, approvals, retrieval citations, model/tool usage, mutations, artifacts, verification and external outcome links. Audit delivery itself is monitored. A run with an audit gap cannot be presented as fully observed.

## Governance gates

Before a pilot:

- threat model and data-protection assessment approved;
- exact EIL ACL and revocation behavior tested adversarially;
- plugin signing/promotion owner selected;
- model/tool egress destinations allow-listed;
- observability capture manifest reviewed with employee/privacy stakeholders;
- incident response and kill-switch exercise completed;
- no production mutation capability enabled.

