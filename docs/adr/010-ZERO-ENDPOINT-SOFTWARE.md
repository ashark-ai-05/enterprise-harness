# ADR-010: No Unapproved Corporate Endpoint Runtime

Status: proposed

## Context

The product owner confirmed that DeepSeek Harness cannot be installed on the corporate laptop because it is unapproved software. The evidence closes the dsh dependency/bundle path, but does not by itself prove whether a separately reviewed EIL-shaped stdio artifact is allowed.

## Decision

The P1 knowledge-pack experiment may use a reviewed first-party extension inside EIL's existing source/lockfile/stdio boundary only after the exact endpoint artifact, dependencies, launch model, client configuration, and proposed change are explicitly approved. This preserves one ACL, validation and audit path and avoids creating an operational service before product value is known; it is not an approval assumption.

The later enterprise harness defaults to a centrally managed service. Already-approved Copilot, Amp, IDE, browser or automation clients reach it through enterprise-authenticated remote MCP/HTTPS. Authoritative tool plugins, credentials, models, durable run state, sandboxes and telemetry remain on approved infrastructure.

DeepSeek Harness is architecture research only. It is not a dependency, fork, bundle, adapter, reference client, local fallback or deployment target.

The endpoint stores no corpus, pack bytes, long-lived credentials or authoritative harness runtime. If an approved client cannot support the required authenticated remote interface, it remains unsupported until an approved vendor/platform path exists.

A minimal stdio transport adapter may be considered only after explicit approval for its pinned artifact, lockfile, dependencies and launch model. It contains no policy, corpus, credentials or execution authority. Fetch-and-run distribution such as `npx ...@latest` and persistent local web servers are prohibited unless separately approved.

## Consequences

- P1 can proceed locally after explicit implementation approval; lightweight confirmation of its in-house review path runs in parallel.
- Shared/team deployment is blocked on EIL per-user identity over HTTP MCP and at least one approved client's authenticated remote integration.
- There is no offline corporate mode and no silent fallback that bypasses policy or observability.
- Long-running work executes server-side and survives client disconnects.
- Corporate proxy, private networking, SSO delegation, service operations and server-side isolation become phase-zero architecture requirements.
- Phase 0 must determine whether EIL itself is approved on the same corporate laptop or operates under an exception/different trust boundary.
- The system sacrifices unrestricted local extensibility for deployability, central revocation, consistent policy and auditability.
