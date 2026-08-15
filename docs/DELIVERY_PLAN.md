# Delivery Plan and Decision Gates

No implementation is authorized yet. This document defines the sequence after approval.

The recommended default is Architecture A: a single-developer, read-only knowledge-pack experiment inside EIL's existing approved boundary. This remains a recommendation until the four questions in `DECISION_REVIEW.md` are confirmed.

## Phase 0 — validate constraints, contracts and product fit

Time-box a design validation using existing EIL and Observability demonstrations.

- Interview 3–5 target teams and select one change workflow.
- Trace identity, ACL, citation, deletion and revocation end to end.
- Specify only the knowledge-pack, resolution-lock, measurement and event-contract deltas needed by P1. Defer capability/workflow schemas.
- Build paper prototypes for the EIL MCP tool shape, manifest review and evidence output. Do not prototype a new CLI or registry.
- Validate the existing EIL toolchain and the exact internal approval path for reviewed first-party changes.
- Measure the baseline corpus/query families, retrieval volume, context size and evaluation cost needed for the falsification trial.
- Threat-model pack scoping, partial views and indirect prompt injection. Defer mutation/sandbox threat modelling until P2 is requested.
- Design the three-arm comparison: A whole authorised corpus, B equivalent inline selectors, C named/versioned pack. Do not attribute A → B scoping gains to the pack abstraction; only B → C and non-author reuse can justify pack machinery.
- Read and test EIL's actual coverage/freshness contracts (`coverage`, source health, evidence bounds) before specifying harness equivalents. Extend those contracts; do not create a second definition of completeness or staleness.
- Record client identity propagation, EIL per-user tokens and HTTP MCP as Phase-2 dependencies; do not put them on the local P1 critical path.
- Confirm directly whether EIL's current source/lockfile/stdio installation is approved on the same corporate laptop, temporarily excepted, or running in another trust boundary. Do not infer this from development use.

**Gate:** scoped packs demonstrate measurable value; approve the product boundary, pack semantics, ownership, pilot workflow, risk controls and success metrics.

**Pre-gate architecture fork:** if EIL's current local execution and proposed extension are formally approved, run the read-only falsification trial as Architecture A behind EIL's existing stdio tool boundary. If not, use Architecture B and wait for the remote identity prerequisites. Do not fund both walking skeletons. See `DEPLOYMENT_OPTIONS.md`.

## Phase 1A — establish the honest inline-filter baseline

Only after explicit approval: extend EIL search with the finest useful inline selectors for Confluence space, Jira project and code repository/path, behind its existing `REGISTRY`/`callTool()` boundary. Compare unscoped arm A with inline-filter arm B. This is independently useful even if packs are rejected.

The existing `documents.hierarchy` data supports the laptop trial without re-ingestion, but it is unindexed. Arm C must apply lock IDs as a predicate on the same FTS/visibility/validity query as A and B; returning the lock's documents instead of ranked search is prohibited. Normal phase-two `get_doc(id)` remains allowed after a pack-scoped result and must compose the shared visibility clause. Latency is usable, provided each predicate and query plan is recorded. Do not add an index speculatively merely to balance the experiment.

**Gate:** inline scoping improves context/setup without material recall loss. If not, stop; packs cannot repair a failed scoping premise.

## Phase 1B — EIL-native pack mechanics and three-arm measurement

Add the smallest content-free knowledge-pack contract, use reviewed Git manifests, and compare A unscoped, B equivalent inline selectors and C the pack. No new runtime, registry service, model loop, tool pack, source mutation, sandbox or approval engine. Ship the shadow comparison and run-level outcome holdout at the same time. This control cannot be reconstructed after packs become the paved road.

**Gate:** zero ACL escapes; complete citations; deterministic evidence identity; policy/telemetry coverage; no material pack recall loss; pack setup is understandable. Do not claim enterprise pack value from a single author.

## Shared-service prerequisite

Architecture A is viable only under explicit approval and does not scale beyond its local single-principal boundary. Architecture B must not deploy until EIL's per-user tokens and HTTP MCP transport are implemented and adversarially verified. This is a dependency on EIL's roadmap, not a harness feature that can be approximated with local ACL filters or endpoint shims.

Before Phase 3, inventory approved clients' authenticated remote MCP/HTTPS support and confirm approved server runtime, artifact registry, deployment platform, private connectivity, SSO/OAuth delegation and corporate proxy paths.

## Phase 2 — Git-distributed multi-developer reuse trial

Distribute reviewed pack manifests through Git while every developer continues to query their own local EIL identity/corpus. Measure use by non-authors, time saved recreating selectors, pack health/rot and outcome guardrails. No shared index or registry service.

**Gate:** meaningful non-author reuse. If reuse stays near zero, keep saved filters as a small EIL feature and stop the enterprise pack platform.

## Phase 3 — team-shared read-only service, only if reuse passes

Implement EIL per-user identity over HTTP MCP, a minimal shared pack catalog, authenticated remote clients and adversarial multi-principal ACL validation. Preserve the same pack contracts; do not add mutations.

**Gate:** P1/P2 meet recall/context/reuse gates; per-user identity is fail-closed; source owners accept the shared operating model.

## Phase 4 — governed execution, only if demanded

Add disposable code workspace, bounded patch/test steps, resumability, approval and evidence bundle. Outputs stay in the sandbox or a draft artifact.

**Gate:** isolation tests, retry/idempotency behavior, unknown-completion handling, kill switch and recovery exercise pass.

## Phase 5 — one governed enterprise mutation

Add one low-risk reviewable mutation, such as creating a draft pull request or Jira comment. Keep merge/deploy outside the harness.

**Gate:** source owner, Security and Risk approve; preview/approval/reconciliation evidence is complete; pilot users confirm value.

## Phase 6 — catalog and organizational scale

Add promotion workflows, tenant quotas, domain ownership, conformance suite, SLOs, regional deployment and cost/capacity controls. Expand workflows only when reusable patterns are proven.

## Pilot success criteria

- 100% of retrieved claims cite an authorized source/version;
- no cross-user, cross-team or cross-tenant ACL failures;
- at least 95% metadata telemetry coverage and explicit gap reporting;
- all consequential actions linked to identity, plan, approval and evidence;
- measurable reduction in time to a reviewable artifact without lower acceptance quality;
- users can explain why a result was retrieved and why an action was allowed;
- platform operating cost and latency fit agreed budgets;
- no individual productivity ranking or sensitive content captured outside policy.

## Decisions required before implementation

1. Confirm the harness boundary and the three-plane ownership model.
2. Confirm manifest-first packs and whether offline snapshot packs are required.
3. Select the pilot workflow and participating domain owner.
4. Select the enterprise artifact/signing registry for plugins and packs.
5. Confirm the policy engine approach and policy owner.
6. Confirm sandbox provider, model gateway and approved MCP topology.
7. Approve the metadata/content capture manifest and retention.
8. Set autonomy tiers and identify the first permitted mutation, if any.
9. Agree SLOs, quotas, budget limits and pilot success thresholds.
10. Confirm the EIL per-user HTTP identity milestone and name its owner/date before planning any shared-index phase.
11. Confirm at least one already-approved client can use the authenticated remote interface without installing harness software.
12. Decide whether a minimal reviewed stdio transport adapter is permitted. This is optional and does not change the managed-service authority boundary.

## Work explicitly excluded until approval

- runtime source files, package manifests and executable scripts;
- CI/CD pipelines or cloud resources;
- connector credentials or live enterprise integrations;
- plugin installation or registry publication;
- EIL/Observability schema changes;
- autonomous mutation behavior.
