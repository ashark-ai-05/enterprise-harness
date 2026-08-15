# Enterprise Information Pack Specification

## Decision

A pack is a **governed, reproducible selection and usage contract over EIL resources**. It is not the EIL ingestion format and normally contains no copied source bodies.

This separates three concepts:

1. **Source ingestion** — EIL connector turns Jira, Confluence, Git or documents into normalized resources and indexes.
2. **Pack build** — publisher selects resource/query rules, pins versions and policies, runs validation, and signs a manifest.
3. **Pack resolution** — harness asks EIL to resolve the manifest for the current principal and purpose, then searches/fetches within the resulting scope.

## Pack types

- **Reference pack:** curated durable resources such as standards and architecture.
- **Live scope pack:** query/filter rules resolved against current EIL state, with a recorded resolution receipt per run.
- **Workflow pack:** reference/live packs plus approved recipes, tools and evaluations.
- **Snapshot pack:** encrypted exported content for explicitly approved offline/disconnected use. It has a short TTL, named recipients, revocation caveat and no silent refresh.

## Illustrative manifest

```yaml
apiVersion: eil.enterprise/v1alpha1
kind: InformationPack
metadata:
  name: payments-change-context
  version: 1.4.0
  owner: team:payments-platform
  tenant: acme
  digest: sha256:...
spec:
  purpose: change-implementation
  classification: confidential
  expiresAt: 2026-11-15T00:00:00Z
  resources:
    - ref: eil://confluence/page/123@sha256:...
    - ref: eil://git/repo/payments/path/src/**@commit:abc123
  liveScopes:
    - source: jira
      filter: project = PAY AND labels = harness-approved
  retrieval:
    lexicalWeight: 1.0
    semanticWeight: 1.0
    maxSnippets: 20
    fetchRequiresExplicitStep: true
  compatibleCapabilities:
    - mcp:github.read@^2
    - sandbox:test@^1
  evaluations:
    dataset: eil://eval/payments-pack-v3
    minimumCitationPrecision: 0.95
  policy:
    requiredGroups: [payments-engineering]
    allowMutations: false
  signatures:
    - issuer: team:payments-platform
      sig: ...
```

The exact schema remains a phase-zero decision. The example records intent, not implementation.

## Build lifecycle

```text
draft → resolve candidate resources → classify → validate ACL/freshness
      → evaluate retrieval → owner/security approval → sign → publish
      → monitor drift/usage → supersede/deprecate/revoke
```

The build receipt records source cursors, resource digests, EIL index generation, connector versions, builder version, policy result, evaluation result and signer. Live scopes additionally record their resolved resource set or query/index receipt for each run.

## Search semantics

- Pack search is EIL search constrained by a pack-derived authorization scope.
- Authorization is evaluated at query and fetch time, not only at publication.
- Search returns cited snippets; full-content fetch is explicit.
- Results identify source, resource version, pack resolution, retrieval strategy and score components.
- Pack dependencies resolve to immutable digests and reject cycles.
- Pack overlays may narrow but never broaden parent policy.
- Cross-pack search de-duplicates by canonical resource/version while retaining all contributing pack IDs.

## Freshness and revocation

Reference packs can be reproducible and stale; live packs can be fresh and change over time. The manifest must declare which property it prioritizes. Runs pin the resolved pack digest and EIL index generation so results remain explainable.

Revocation blocks new resolutions immediately. Previously fetched content follows retention policy; snapshot packs cannot guarantee remote deletion, so their use must be exceptional and time-bounded.

## Supply-chain controls

- signed manifests and immutable digests;
- approved publisher identities and protected promotion channels;
- dependency lock and SBOM for executable workflow content;
- static checks for secret references, broad scopes and prohibited tools;
- retrieval evaluation and adversarial ACL tests before promotion;
- reproducible build receipt and rollback to previous digest;
- quarantine for compromised publishers or dependencies.

## Open questions for phase 0

1. Should live scopes pin every resource ID or pin the query plus EIL index generation?
2. Which existing enterprise artifact registry should store manifests and signatures?
3. What is the maximum acceptable time between source ACL revocation and EIL enforcement?
4. Which pack fields are centrally governed versus domain-owner configurable?
5. Are encrypted offline snapshots required in the first year?

