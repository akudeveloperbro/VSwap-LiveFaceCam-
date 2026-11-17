# GPU Provider Abstraction

**Status:** Provider-neutral design and evaluation plan; no provider selected or integrated. Updated 2026-11-18.

## Boundary

Only backend orchestration calls provider adapters. The browser selects a product-level mode, not a provider API, credential, machine image or arbitrary endpoint. The Python engine is deployed separately from provider SDKs and implements inference/session contracts. Provider-specific resource identifiers remain private backend metadata.

```text
Orchestrator → WorkerProvider
  provision(spec) → WorkerLease
  inspect(lease) → Health/State
  terminate(lease) → TerminationResult
  listCapabilities(region) → CapabilitySnapshot
```

The interface should cover idempotent provision, readiness/health, capacity/capability discovery, endpoint/session attachment, termination, bounded retries, provider error normalization, cost/usage observations and correlation IDs. Separate persistent workers for real-time sessions from ephemeral batch capacity if lifecycle semantics differ. Never require an adapter to expose a raw provider SDK object to the engine or UI.

## Candidate provider evaluation (not researched or verified)

The following are investigation candidates only. No current price, latency, GPU inventory, availability, persistence property, compliance statement or API fitness is asserted here; all can change by region and date.

| Candidate | Evaluate | Key questions |
|---|---|---|
| RunPod | GPU availability/pricing, persistent GPU endpoints, custom container/deployment flexibility, startup/recovery APIs, network paths and service limits. | Can a long-lived interactive worker be reliably addressed with bounded startup time, supported health checks and predictable billing? |
| Vast.ai | Marketplace capacity/pricing, GPU/host variability, container and networking controls, persistent allocation semantics, reliability and support. | Can variability and host/network trust be bounded enough for private, low-latency user sessions? |
| Modal | GPU availability/cost, container and function/worker lifecycle, persistent process support, ingress/egress and streaming suitability. | Which runtime pattern supports a session-affine stream rather than short request/response invocation? |
| Replicate | Model deployment/versioning, API latency and billing, custom model/runtime support, streaming/session semantics and data handling. | Is the product's model and long-lived bidirectional real-time session supported, or is this primarily a batch/inference API fit? |

## Evaluation scorecard

Before selecting a provider, obtain date-stamped official pricing and test the same engine/image/workload in comparable regions. Score: total cost (GPU, idle time, storage, egress, startup), cold/warm readiness, p50/p95 round-trip and inference latency, sustained FPS/jitter/drops, session duration limits, actual GPU/model availability, failure rate/recovery, networking/WebRTC compatibility, container/deployment control, observability, secret management, regions/data terms, security/compliance evidence, support, quotas and lock-in/exit effort. Define weights and workload before scoring; do not select from list prices alone.

## Adapter safety and test requirements

- All provisioning inputs come from server policy allowlists; validate GPU class, image digest, region, limits and maximum cost.
- Use backend-only credentials from managed secret storage; rotate and scope them. No provider token in client config or logs.
- Provision/terminate must be idempotent; reconcile orphaned leases and use TTL/lease expiry.
- Normalize provider states into internal lifecycle states and preserve sanitized provider error codes for operations.
- Contract-test adapters with a fake provider; integration tests against a sandbox account are a later approved step, not part of documentation-only work.
- Track actual resource consumption and chargeback estimates; reconcile provider invoices with application usage.

## Decision gate

No provider may be called “selected” until a documented scorecard, representative workload benchmark, privacy/security review, expected utilization/cost model, operational runbook and explicit approval are complete. A provider abstraction does not guarantee portability; container, network, model runtime and streaming constraints must be tested per adapter.
