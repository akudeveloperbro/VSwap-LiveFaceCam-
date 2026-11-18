# VSwap Master Implementation Blueprint

**Status:** Phase 3 engineering specification; proposed, not an implementation authorization. Source code and infrastructure remain unchanged. Updated 2026-11-18.

## 1. Goal and source of truth

Build a consent-led, web-first face transformation product with separable web UI, independent API/control plane, reusable Python engine, replaceable GPU provider adapters, and a distinct real-time media plane. The current repository is a Python/PySide6 desktop application with local camera/image/video processing; it has no web frontend, FastAPI service, PostgreSQL schema, WebRTC service, cloud worker, or provider adapter. See [`../WEB_PLATFORM_ARCHITECTURE.md`](../WEB_PLATFORM_ARCHITECTURE.md) and [`../REALTIME_ENGINE_IMPLEMENTATION_PLAN.md`](../REALTIME_ENGINE_IMPLEMENTATION_PLAN.md).

All claims in this blueprint are one of: **Verified** (current repo observation), **Proposed** (architecture/spec), or **Unverified** (requires evidence). No provider is selected. RunPod, Vast.ai, Modal and Replicate are evaluation candidates only. The WhatsApp objective is not a compatibility claim.

## 2. Recommended target architecture

- **Frontend:** Next.js App Router, React, TypeScript, Tailwind, shadcn/ui; mobile-first. The browser owns presentation, explicit camera permission and local preview, but not identity authority, orchestration, private storage, provider credentials or inference.
- **Independent backend:** Python + FastAPI is the leading candidate because the engine is Python; PostgreSQL is recommended for durable relational state. SQLAlchemy 2.x plus Alembic is a candidate, pending team confirmation and migration review. Use Redis only if queue/lease/presence requirements justify it; PostgreSQL remains source of truth. Private object storage is required before remote media processing.
- **AI engine:** current LiveFaceCam repository is a foundation, not a headless service. Keep GUI/camera/provider dependencies behind adapters; extract a versioned engine API only after a license and dependency audit.
- **Batch:** signed private upload → API-created asynchronous job → bounded worker → private output asset → authorized result retrieval.
- **Live:** camera acquired after user action → HTTPS session admission + authenticated WebSocket signaling/control → WebRTC media path → persistent worker session → transformed return track. Never REST per frame.
- **Compute:** independently verified browser-local CPU option; cloud GPU via server-side provider adapters; explicit-consent hybrid routing only after both modes work independently. Initial provider assessment favors benchmarking RunPod persistent Pods and Modal session-capable services for live, with serverless paths from RunPod/Modal/Replicate for batch; Vast.ai remains a cost/capacity experiment until host variability and isolation are evidenced. No selection or pricing claim is made; see [worker assessment](GPU_WORKER_IMPLEMENTATION_SPEC.md).
- **Native/WhatsApp:** Windows/macOS OS virtual cameras and Android/iOS system-camera support are separate components and release gates. No client compatibility is verified; see [test matrix](TESTING_AND_ACCEPTANCE_PLAN.md).

## 3. Boundaries and core contracts

Frontend consumes versioned capability/job/session/media/usage APIs and reports backend truth. API validates identity, consent, ownership, limits, entitlements, idempotency and lifecycle. Orchestrator selects internal worker/provider from allowlists. Worker runs pinned engine/model versions and emits bounded, aggregate telemetry. Engine knows nothing about provider credentials or billing. Object storage is private and access is scoped and short-lived. WebSocket carries signaling/status/metrics; WebRTC carries real-time media; batch media uses object storage and jobs.

Proposed lifecycle states: `queued`, `starting`, `running`, `completed`, `failed`, `cancelled`, `expired`, with `degraded` for a live session where output continues below policy targets. Stable error codes, retryability, request correlation IDs and timestamps are required. Preview-only, waiting-for-worker, and transformed output must never share one “live” state.

## 4. Security/privacy invariants

Explicit authorization/consent is required for likeness and target media. Remote fallback cannot happen silently. All records/media are user/tenant scoped. Provider credentials remain backend-only. Enforce server-side decoded media validation, file/duration/resolution/concurrency/cost limits, abuse controls, rate limits, private storage, encryption in transit/at rest, bounded retention/deletion, audit metadata minimization, job isolation and incident response. Do not persist face embeddings by default. Synthetic-media disclosure and model rights require policy review. No call-audio capture or WhatsApp security bypass.

## 5. Implementation roadmap

| Milestone | Objective / affected area | Dependencies | Acceptance and tests | Complexity / blockers |
|---|---|---|---|---|
| M0 Architecture decisions | Finalize scope, browser/OS targets, consent, retention, auth, costs, model/license and support claims; docs only. | Product, legal/privacy, engineering approvals. | Signed decision register entries; explicit unresolved list. | M; multiple policy/ownership decisions. |
| M1 Frontend foundation | Create route shell/design system and capability-driven states in `frontend/`; no false processing. | API contract or capability stub explicitly labeled non-production; choose preview auth strategy. | Mobile 414×765; accessible loading/error/empty/offline/unsupported states; browser tests; no mock success. | M; API endpoint and design system decisions. |
| M2 Backend foundation | FastAPI versioned service, config, health/readiness, auth boundary and contract tests in `backend/`. | M0, language/tooling, auth design, deployment owner. | OpenAPI contract, auth/health tests, structured errors, CI; no inference endpoints until backend baseline. | M/L; auth/session and hosting. |
| M3 Identity + data | PostgreSQL schema/migrations, user/session, ownership, audit/consent, private media metadata. | M2, retention/tenant/auth decisions. | Migration up/down, ownership denial, retention/deletion, backup/restore test. | L; deletion/account lifecycle and data policy. |
| M4 Batch media/jobs | Private upload, image job/status/result; bounded queue; API and worker contracts. | M3, storage choice, engine/license feasibility. | Oversize/malformed rejection, idempotency, cancel/failure/cleanup, private result authorization, real inference evidence. | L; model licensing and headless engine. |
| M5 Engine integration | Headless reusable engine seam; preserve desktop behavior; test adapter. | M4 contract, engine audit, model distribution approval. | GUI-independent tests; output quality baseline; pinned model/runtime; deterministic resource close. | L; PySide coupling, model terms, GPU memory. |
| M6 Local CPU feasibility | Browser-local WASM/lightweight model spike; do not confuse with desktop CPU. | Model/license and browser support review. | Device matrix, download/memory/thermal/FPS/quality measurements; mark unsupported if failing. | L; model runtime and performance uncertainty. |
| M7 First cloud provider | Implement one provider only after scorecard and approval; GPU worker lifecycle and batch job first. | M4/M5, spend cap, provider/security approval. | Sandboxed provision/health/terminate, provider fault tests, cost reconciliation, no leaked credentials. | L; provider selection, networking and cost. |
| M8 WebRTC live | Persistent streaming session, auth signaling, media gateway/worker, metrics/recovery. | M5, M7 or verified local stream capability, topology decision. | Sustained 720p30 target protocol; frame age/FPS/jitter/drops; network/failure/consent tests. | XL; WebRTC topology, TURN, worker affinity. |
| M9 Native camera + compatibility | OS virtual camera/agent and WhatsApp test matrix, separately from web app. | M5, OS API/legal/distribution review. | Remote participant sees transformed frames on named OS/client build; sustained call and shutdown tests. | XL; drivers/signing/OS limits. |
| M10 Optimization/productization | 1080p60 investigation, auto hybrid, billing, abuse operations and scaling. | Stable M4/M8/M9 evidence, usage/cost/retention decisions. | No performance claim without repeatable metrics; load/security/operations review. | L/XL; production economics and compliance. |

T-shirt sizes are relative engineering complexity, not calendar commitments.

## 6. Quality and release gates

Use [`TESTING_AND_ACCEPTANCE_PLAN.md`](TESTING_AND_ACCEPTANCE_PLAN.md). Core measurable live target is proposed 720p/30 with p95 capture-to-return frame age ≤100 ms over 10 minutes on approved reference hardware, with reported FPS, jitter, drops and memory. This is not a current baseline or a call-network latency claim. 1080p/60 remains a future investigation. A real transformed output—not a preview or WebRTC connection—is required for inference acceptance.

## 7. First executable milestone

After approval of M0, first code milestone is **M1 Frontend foundation**: a responsive route shell and capability-driven UI backed only by an explicitly non-production contract fixture or unavailable state, with no fake users, jobs or successful swaps. The first end-to-end functional slice should follow M2–M4 with authenticated private image upload/job/result before live streaming.

## 8. Approval gate

Before source-code changes, the owner must explicitly approve: (1) M0 scope and first milestone, (2) auth/session approach, (3) media consent/retention/region policy, (4) model/license deployment feasibility, (5) whether cloud GPU and any paid provider may be evaluated/provisioned, (6) budget ceiling, and (7) whether implementation may begin in a feature branch. Approval of this documentation is not approval to deploy resources or claim WhatsApp compatibility.

## Supporting specifications

- [Monorepo structure](MONOREPO_STRUCTURE.md)
- [Database](DATABASE_IMPLEMENTATION_SPEC.md)
- [API](API_IMPLEMENTATION_SPEC.md)
- [Frontend](FRONTEND_IMPLEMENTATION_SPEC.md)
- [Backend](BACKEND_IMPLEMENTATION_SPEC.md)
- [Engine integration](AI_ENGINE_INTEGRATION_SPEC.md)
- [GPU workers](GPU_WORKER_IMPLEMENTATION_SPEC.md)
- [WebRTC](WEBRTC_IMPLEMENTATION_SPEC.md)
- [Testing](TESTING_AND_ACCEPTANCE_PLAN.md)
- [Decision register](ARCHITECTURE_DECISION_REGISTER.md)
