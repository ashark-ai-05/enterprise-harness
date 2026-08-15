# DeepSeek Harness Reference Assessment

## What to retain

DeepSeek Harness uses a plugin tree in which model adapters, tools, persistence, sandboxing, approvals, telemetry, sessions and the agent loop are replaceable services. Profiles and bundles compose ordered configuration layers. Durable session events are the source of model-visible context, while live events intercept work in flight. Capability seams separate service definitions, providers and consumers.

These ideas fit the enterprise harness:

- explicit capability seams;
- composable profiles/bundles;
- durable event-sourced run history;
- replaceable model, tool, sandbox and persistence providers;
- tool execution hooks before and after every call;
- scoped capabilities per agent/run;
- headless and interactive surfaces over one runtime.

Source: [DeepSeek Harness architecture](https://github.com/deepseek-ai/deepseek-harness/blob/master/docs/architecture.md), reviewed 2026-08-15.

## What must change for enterprise use

| Reference idea | Enterprise adaptation |
|---|---|
| Everything is an in-process-composable plugin | Trust-tier plugins; I/O and mutations run out of process behind non-bypassable gateways |
| Session log is the main durable truth | Harness run state is operational truth; Observability is durable cross-product evidence; systems of record own outcomes |
| Local profile/home patches | centrally signed profiles plus tenant/team overlays that may narrow, never weaken, policy |
| Tools registered into prompt | expose only post-policy schemas for the current step/principal/purpose |
| Plugin unload reverses effects | prefer immutable versioned runs and controlled restarts; hot unload is not needed initially |
| Model-visible context derives from session | EIL-sourced context derives from cited resource references and authorization receipts |
| General agent loop | typed, budgeted workflow graph with explicit approvals and verification |

## Critical conclusion

“Everything is a plugin” is a valuable composition principle but a dangerous security boundary. Enterprise extensibility must sit inside a small invariant kernel: identity binding, policy admission, credential brokering, isolation, idempotency, evidence emission and audit cannot be replaced by ordinary plugins.

Likewise, the harness should not absorb EIL. Knowledge ingestion and search have different scaling, governance and freshness semantics from workflow execution. Packs bridge the products through signed manifests and resolution receipts, not copied indexes.

## Build, extend or fork

The recommendation is **runtime-neutral contracts, proven first with a dsh bundle/adapter**.

- Do not fork dsh: it is a large developer-preview codebase that explicitly expects compatibility-breaking changes.
- Do not make dsh the enterprise contract: existing IDE agents, CI jobs and future runtimes should be able to use the same governed packs and workflows.
- Do reuse dsh: its agent loop, session log, web/headless profiles, tool pipeline, sandbox and telemetry seams make it a strong reference host and a fast way to validate the product.

The phase-zero spike should answer whether corporate installation, dependency approval and proxy constraints permit a dsh-based reference client. A negative answer changes the adapter, not the pack/workflow/policy contracts.
