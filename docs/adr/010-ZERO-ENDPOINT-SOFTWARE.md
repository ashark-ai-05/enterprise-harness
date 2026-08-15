# ADR-010: Zero New Corporate Endpoint Software

Status: proposed

## Context

Corporate laptops cannot install DeepSeek Harness or other unapproved runtime software.

## Decision

Enterprise Harness is a centrally managed service. Already-approved Copilot, Amp, IDE, browser or automation clients reach it through enterprise-authenticated remote MCP/HTTPS. New harness code, plugins, EIL access, credentials, models, tools, run state, sandboxes and telemetry remain on approved infrastructure.

DeepSeek Harness is architecture research only. It is not a dependency, fork, bundle, adapter, reference client, local fallback or deployment target.

The endpoint stores no corpus, pack bytes, long-lived credentials or harness runtime. If an approved client cannot support the required authenticated remote interface, it remains unsupported until an approved vendor/platform path exists.

## Consequences

- The first corporate pilot is blocked on EIL per-user identity over HTTP MCP and at least one approved client's authenticated remote integration.
- There is no offline corporate mode and no silent fallback that bypasses policy or observability.
- Long-running work executes server-side and survives client disconnects.
- Corporate proxy, private networking, SSO delegation, service operations and server-side isolation become phase-zero architecture requirements.
- The system sacrifices local extensibility for deployability, central revocation, consistent policy and auditability.
