# Hybrid Processing Architecture

**Status:** Proposed modes and routing policy; no browser CPU or cloud orchestration implementation is verified. Updated 2026-11-18.

## Modes

### Local CPU (browser-local investigation)

Candidate flow: user-authorized browser camera/file input → capability probe → supported lightweight model/runtime → local transformed preview. Investigate WebAssembly first and WebGPU only where supported and appropriate. The existing Python desktop CPU inference path does not establish browser feasibility. Measure model licensing, download size, memory, thermal impact, browser support, privacy, FPS and output quality. Do not call preview-only or untransformed frames face-swapped output. Unsupported device/model combinations must fail closed with a clear explanation.

### Outsourced cloud GPU

Candidate flow: consented upload or WebRTC media session → backend admission and authorization → persistent worker → reusable engine session → transformed output/progress → deterministic teardown. Image/video jobs are asynchronous and resumable; real-time video uses a persistent media session, not REST per frame. Cloud mode requires visible disclosure of remote processing, data flow, provider region, retention and cost/usage implications.

### Automatic hybrid

Policy may select local processing when a tested local capability is available and meets the requested quality/resource profile; otherwise it may select cloud only when the user has explicitly allowed remote processing and backend/provider capacity and budget checks pass. It must not silently move a camera stream or stored media to the cloud. Offer user-visible mode status and a way to stop remote processing.

## Routing inputs and policy

Inputs may include explicit privacy preference, requested operation (image/video/live), model and output-size capability, browser/device capability, measured local performance, network RTT/bandwidth/loss, worker/provider health, queue delay, regional policy, entitlement/quota, and user cost ceiling. The policy must be deterministic and auditable for a given policy version; do not expose raw provider internals unnecessarily.

Proposed selection sequence:

1. Authorize user, consent, operation, media and quotas.
2. Resolve requested capability and policy restrictions (model/license/region/retention).
3. If local-only is selected, test local runtime/model support and run locally or report unsupported; never upload as fallback.
4. If cloud is permitted, check service capacity, predicted queue/session admission and cost budget; ask for confirmation if estimates exceed the previously accepted threshold.
5. Pin mode/provider/engine/model version for the session/job; do not change engines mid-stream without an explicit renegotiation and visible state.
6. On failure, retry only bounded transient errors. Cloud-to-local fallback is allowed only if the local path is already supported and user-approved; local-to-cloud fallback requires explicit prior remote consent.
7. Emit reason codes and aggregate outcome metrics without recording media or embeddings.

## Failure and resource controls

- Local: permission denied, unsupported browser/API/model, memory pressure, thermal throttling, tab backgrounding, model download failure and low FPS.
- Remote: no capacity, provider outage, worker startup/model load timeout, WebRTC disconnect, overloaded inference, quota/cost ceiling, object-storage failure and region restriction.
- Bound queue depth and frame age; drop stale frames rather than accumulate latency. Provide explicit degraded/paused/failed states.
- Enforce maximum job duration, resolution, frame rate, input size, session duration and concurrent allocations.
- Keep circuit breakers and per-provider budgets in backend orchestration; do not let clients choose arbitrary provider IDs or worker URLs.
- Define idempotent cleanup for sessions, media, worker leases and usage events.

## Evaluation gates

Browser CPU path: model/license review, browser/device matrix, startup/download footprint, memory, sustained thermals, quality and p50/p95 frame age. Cloud path: provider comparison, cold/warm startup, session latency, capacity, failure/recovery, security and cost per processed minute/job. Hybrid path: demonstrate no network transfer without consent, deterministic policy decisions, bounded retries and accurate user-visible mode/reporting. Targets in `PERFORMANCE_AND_LATENCY_TARGETS.md` remain proposed and do not yet describe a web baseline.
