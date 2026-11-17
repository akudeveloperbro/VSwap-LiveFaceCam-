# VSwap Web Implementation Roadmap

**Status:** Proposed sequence for approval; no web implementation authorized or performed. Updated 2026-11-18.

## Goal

Deliver a web-first, consent-led VSwap product while keeping the browser UI, independent backend, reusable Python engine and GPU infrastructure separate. Preserve the current LiveFaceCam functionality and repository as the engine foundation. This roadmap sequences investigation and approval before implementation; it does not authorize source changes or paid infrastructure.

## Phase 0 — Architecture/product approval

Resolve MVP operations (image, video, live), local CPU feasibility, cloud GPU inclusion, launch OS/browser/client matrix, consent and abuse policy, media retention/regions, model/license constraints, authentication, latency/quality targets, billing scope and cost ceiling.

**Exit:** approved architecture decision record, scope, trust boundaries, data lifecycle and measurable acceptance criteria. No production integration or provider commitment.

## Phase 1 — Evidence and feasibility spikes (separately approved)

1. Inventory and license-review engine dependencies/models; verify whether the existing processing code can be packaged headlessly without modifying its behavior.
2. Test browser-local WebAssembly/runtime and model feasibility on a small declared device/browser matrix.
3. Compare RunPod, Vast.ai, Modal and Replicate using date-stamped pricing, representative workload, startup/streaming/network measurements, availability, security/data terms and operational fit.
4. Validate WebRTC media topology and signaling/worker lifecycle in an isolated prototype only after privacy and infrastructure approval.
5. Research WhatsApp Desktop/Web/mobile paths independently; test only supported camera selection and documented OS APIs.

**Exit:** evidence report with failures and blockers, not vendor marketing claims; approve or reject each capability path.

## Phase 2 — Contract and platform foundation

Design API/version/auth/session/error contracts, identity and tenant model, PostgreSQL schema, private object storage and retention, job lifecycle, event delivery semantics, audit/usage events, provider interface, worker protocol, secrets, quotas and operational runbooks. FastAPI + PostgreSQL remain candidates pending approval; broker choice should follow measured queue/stream requirements.

**Exit:** reviewed threat model and contracts, migrations and ownership boundaries approved before dependent application code.

## Phase 3 — Web UI foundation (no fake processing)

Build the modular mobile-first dashboard and design system around backend capability discovery. Implement settings/navigation, real API loading/error/empty states, consent/camera permission affordances and capability-gated screens. Dashboard performance, billing, source management, Live Studio and image/video submission may not imply that unavailable inference is working. Use explicit unavailable/coming-later states where service capabilities do not exist.

**Exit:** browser tests verify mobile layout, accessibility, permission denial, API outage, unavailable capability and no false-success states.

## Phase 4 — Authenticated batch processing (recommended initial inference slice)

If approved: implement authentication/authorization, private upload, validation, asynchronous image job lifecycle, worker integration, progress and private result retrieval. Then extend to bounded video jobs after duration/size/cost policy and cleanup are validated. Keep model/engine version pinned; do not make frontend responsible for inference or provider credentials.

**Exit:** end-to-end authorized tests; job idempotency/cancellation, tenant isolation, deletion, failure states, license review and measured cost/quality.

## Phase 5 — Live streaming

Only after a working engine worker and security boundary: establish WebRTC signaling/media topology, persistent GPU session, bounded low-latency queues, reconnect/teardown behavior, explicit remote-processing consent, and end-to-end metrics. Validate mobile browser behavior and network failure separately. Do not process each frame over REST.

**Exit:** sustained real-device benchmarks with p50/p95 frame age, FPS, jitter, drop rate, cost and recoverability, plus consent/security review.

## Phase 6 — Hybrid routing and product completion

Add automatic policy only when local and cloud modes independently pass their acceptance criteria. Add account management, usage/cost controls and provider health UX. Add subscription/billing surfaces only with approved entitlement and payment integration. Add WhatsApp/native compatibility only after the corresponding OS/client matrix passes end-to-end calls.

**Exit:** deterministic, consent-aware routing; verified support matrix; operational alerts, abuse response, retention/deletion and support runbooks.

## Risks and blockers

- Existing source is a desktop application, not a verified importable headless service.
- Browser CPU model feasibility and applicable model licenses are unresolved; current README describes non-commercial/model-specific restrictions.
- No backend/auth/database/storage/worker/provider integrations were found in the inspected tree.
- GPU provider pricing, availability, persistence, streaming latency and data terms are not yet evaluated.
- WebRTC cannot make the web app a system virtual camera or provide an arbitrary transformed camera track to WhatsApp Web.
- WhatsApp Desktop/Web and Android/iOS compatibility is untested and may require native OS components.
- Billing, data retention, consent/abuse controls, privacy region and operational cost limits are unresolved.

## Acceptance principles

No fake processing, sample data represented as real account results, optimistic success, fabricated metrics or provider selection claims. Empty/loading/offline/error/permission-denied/degraded states are first-class. Every remote media path requires explicit consent and bounded retention. No implementation phase begins before the relevant exit decision is approved.
