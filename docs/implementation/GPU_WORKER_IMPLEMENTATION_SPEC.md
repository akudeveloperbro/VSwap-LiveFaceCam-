# GPU Worker Implementation Specification

**Status:** Provider-neutral proposal; no provider chosen, code written, or GPU provisioned. See [`../GPU_PROVIDER_ABSTRACTION.md`](../GPU_PROVIDER_ABSTRACTION.md).

## Worker roles

Separate persistent interactive sessions from batch job execution where resource and lifecycle needs differ. A worker owns a pinned engine/model runtime, bounded input/output buffers, inference/rendering, health and aggregate telemetry. Backend owns identity, consent, scheduling, quotas, provider credentials and durable state. Worker access is an expiring, narrowly scoped lease; it is not a public API.

## Lifecycle protocol

`requested → provisioning → booting → loading_model → ready → busy/degraded → draining → terminated`; terminal error path `failed`. Provision/terminate idempotently. Worker reports readiness only after runtime, model digest and GPU capability checks. Sessions/jobs are separately stateful and must be isolated. Lease heartbeat expiry triggers reconciliation and bounded cleanup. Startup, model load, task duration, memory, queue depth, frame age and retries all have hard limits.

## Runtime packaging

Reproducible container image pinned by digest; pinned OS/CUDA/driver compatibility and engine/model versions; non-root user; read-only root where practical; minimal capabilities; no Docker socket; outbound network allowlist; scoped short-lived object access; secrets injected from managed secret store. Model weights are mounted/fetched only from approved source and verified by checksum. License/redistribution decision is a blocker before bake-in.

## Work protocols

- Batch job: worker receives authorized asset reference and validated options, pulls to encrypted/ephemeral scratch, processes bounded chunks, uploads private output, emits progress, deletes temp input/output per retention policy.
- Live session: persistent session-affine media pipeline; WebRTC/media gateway owns negotiated transport; worker gets a bounded frame stream through approved internal protocol and returns frames with capture timestamps. Do not use REST-per-frame. Define backpressure and latest-frame drop rules.

## Candidate provider assessment (initial, not a selection)

This is a fit assessment, not a pricing quote or verified benchmark. Current GPU inventory, regional availability, exact pricing, startup and latency must be checked against dated official terms and a common workload before selection.

| Candidate | Apparent fit to investigate | Main trade-offs / open questions |
|---|---|---|
| RunPod | Serverless workers look suited to bursty batch jobs; dedicated Pods offer a persistent-GPU path to investigate for live sessions. | Scale-to-zero versus always-on/idle costs; cold model load; session affinity, ingress and worker health/recovery; regional capacity and service limits. |
| Vast.ai | Marketplace instances and API-driven provisioning may offer flexible GPU choices and competitive spot-like capacity for experiments/batch. | Host/GPU/network variability, trust boundary, availability guarantees, persistent allocation and session reliability; requires stronger host isolation and qualification before private live media. |
| Modal | Managed Python/container and GPU execution may simplify API-driven batch and autoscaling; investigate web endpoints for live transport. | Confirm long-lived bidirectional streaming/session affinity, timeout/concurrency/scale behavior, cold starts, regional network/TURN path and persistent worker economics. |
| Replicate | Hosted, versioned model inference/deployments can simplify batch model serving and API operation. | Determine custom model/runtime compatibility, cold starts, continuous session/WebRTC support, deployment idle billing, data path and whether its request/response model fits low-latency live video. |

**Preliminary recommendation:** compare RunPod dedicated persistent workers and Modal session-capable GPU services for live, and include Replicate/Modal/RunPod serverless paths for batch; evaluate Vast.ai as a cost/capacity experiment rather than a production default until host variability, isolation and support requirements are satisfied. This is a test shortlist only, not an approved provider choice. All four require common benchmark and privacy/security review. Pricing changes frequently; record official pricing date and compute+idle+storage+egress+retry cost per successful minute/job. Official starting points: [RunPod docs](https://docs.runpod.io/), [Vast.ai docs](https://docs.vast.ai/), [Modal docs](https://modal.com/docs), [Replicate docs](https://replicate.com/docs).

## Provider-neutral interface

Orchestrator invokes validated `provision`, `inspect`, `terminate`, `capabilities` operations. Provider resource IDs and SDK errors stay in adapter/admin scope. Apply quotas, region/GPU allowlists, spend ceilings, TTL, circuit breaker, cost reconciliation and orphan cleanup. Provider choice is not user-controlled arbitrary metadata. No RunPod/Vast.ai/Modal/Replicate integration is approved by this specification.

## Failure, privacy, and acceptance

Test no capacity, slow boot, model mismatch, OOM/GPU reset, worker crash, storage timeout, queue saturation, lost control plane, duplicate commands, revoked consent, expired lease and malicious media. Ensure bounded recovery and visible terminal state; no silent provider switch mid-session. Clear session data and scratch; verify deletion. Record aggregate startup/inference/frame-age/FPS/drop/error/cost metrics with no media, embedding, raw path or credentials. Before launch require isolation tests, sustained load/fault tests, security review, cost ceiling, operations alerts/runbook and approved data-region/retention terms.
