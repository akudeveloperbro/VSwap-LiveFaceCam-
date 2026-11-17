# Cloud GPU Worker Architecture

**Status:** Proposed service design; no cloud workers or provider deployments exist in this repository. Updated 2026-11-18.

## Worker responsibilities

A worker hosts the reusable Python engine in a pinned runtime/container and owns bounded preprocessing, model/session initialization, inference, compositing, output and aggregate health metrics. It does not own account identity, billing authority, provider selection policy or durable user records. The orchestrator grants a short-lived session/job capability; worker access is narrowly scoped and expires.

The engine boundary should expose explicit `load(model/runtime)`, `open_session(config)`, `process_frame(frame, timestamp)`, `process_asset(input, options)`, `health()`, and `close_session()` semantics, with versioned schemas and structured capability/errors. This is a target interface, not a claim that current `modules/` provides these methods. Keep the AI engine repository provider-neutral; a worker image/adaptor connects it to infrastructure.

## Lifecycle

1. **Admission:** authenticate/authorize; verify consent, model rights, media limits, region, quota and cost ceiling.
2. **Allocate:** orchestrator selects a provider/region/worker class through the provider abstraction and creates an expiring lease.
3. **Boot/readiness:** worker starts, verifies runtime/model integrity and reports capability/health; enforce startup and model-load deadlines.
4. **Session/job:** pin engine/model versions; isolate user/session state; maintain bounded input/output queues and per-session resource limits.
5. **Observe:** report ready/busy/degraded/failed and aggregate timings, queue age, GPU memory/utilization if available, FPS, drops and errors. Never report inferred hardware telemetry as measured.
6. **Drain/teardown:** stop accepting work, finish or cancel within a bounded grace period, clear per-session media/embeddings, release engine resources and terminate/return worker per policy.
7. **Reconcile:** periodically find expired leases/orphans, close them and reconcile provider usage against application records.

Persistent warm workers can reduce model-load latency but incur idle cost and increase the sensitivity of session isolation. Decide pool size and scale-to-zero behavior from measured demand and budget, not assumptions.

## Real-time vs batch

- **Live:** session-affine, persistent bidirectional media stream; WebRTC terminates at a dedicated media/session boundary or worker-compatible gateway. Do not send one REST request per frame. Define congestion control, frame dropping, keyframe/session recovery and output synchronization.
- **Image/video jobs:** asynchronous job queue; private object storage; worker downloads only authorized inputs, processes in bounded chunks, writes an output object and emits state/progress. Reuse a loaded worker/model where efficient without mixing user state.

## Isolation, privacy and failure handling

- Use separate processes/containers and least-privilege filesystem/network permissions; avoid cross-tenant in-memory media reuse and clear buffers/session state on close.
- Do not mount broad object-store credentials into workers; use job-scoped short-lived access. Restrict egress and validate assets before decoding.
- Limit input dimensions/duration/frame rate, concurrency, memory, GPU time, queue depth and session duration. Terminate runaway tasks.
- Worker crash, OOM, model incompatibility, GPU reset, provider interruption and lost control connection must move the job/session to explicit failed/degraded states and trigger bounded policy-based recovery.
- Never resume an interactive stream on a new provider/worker invisibly. A reconnect may negotiate a new session; disclose interruption and preserve user control.
- Define retention and deletion for originals, temporary files, outputs, caches, logs and backups before deployment.

## Observability and acceptance

Measure worker-ready time, model-load time, queue wait, inference/composite durations, capture-to-output age where source timestamps exist, FPS/jitter/drops, job duration, error/retry counts, resource use, cost per session/minute/job and cleanup lag. Metrics must be aggregate and must not contain frames, face embeddings, source paths or credentials.

Before launch: fault-injection tests for startup timeout, worker death, slow inference, queue saturation, provider timeout, lost WebRTC path and orphan cleanup; cross-user isolation tests; deletion/retention checks; measured cost and performance under representative authorized workloads.
