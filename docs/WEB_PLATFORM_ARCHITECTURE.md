# VSwap Web Platform Architecture

**Status:** Target architecture proposal; implementation has not started. Updated 2026-11-18.

## Goal and decision

VSwap is moving to a web-first product. The v0-built web interface owns account-facing UX and control surfaces; independently operated backend services own identity, authorization, orchestration, durable records and media access; the existing LiveFaceCam repository remains the starting point for a reusable Python AI engine; external providers may supply GPU capacity behind an adapter. No inference or GPU credentials belong in the browser or the Next.js/Vercel frontend runtime.

The architecture is intentionally staged. A browser dashboard, a camera preview, or a WebRTC connection is not evidence that face transformation is implemented. Every capability must report its actual state and fail honestly when a backend/worker capability is absent.

## System map

```text
Browser (Next.js / v0 UI)
  ├─ HTTPS control-plane API ───────────> Independent API (proposed FastAPI)
  │                                         ├─ PostgreSQL: users, sessions, jobs, usage, provider metadata
  │                                         ├─ private media object storage (provider TBD)
  │                                         ├─ job/event broker (only if required; TBD)
  │                                         └─ orchestration / policy / provider adapters
  ├─ WebSocket: authenticated status/events <┘
  └─ WebRTC media ─────────────────────> Streaming gateway / session service (proposed)
                                              └─ persistent GPU worker session
                                                   ├─ provider adapter → RunPod / Vast.ai / Modal / Replicate (evaluation only)
                                                   └─ versioned Python processing engine derived from LiveFaceCam

Browser-local CPU path (experimental, separate capability): camera → supported lightweight local model/WASM runtime → browser preview
```

The diagram is a proposal, not a deployed topology. The signaling/control plane and the real-time media plane are distinct. WebSocket is for control, signaling and coarse progress events; it is not the per-frame video transport. WebRTC is the proposed interactive video transport. Image/video batch processing uses upload plus asynchronous job APIs, not one REST request per frame.

## Component responsibilities

- **Web frontend:** dashboard, Live Studio, accessible camera permission and preview flow, source-face/media management, image/video job controls, processing-mode and provider preferences, measured performance display, settings and billing surfaces. These surfaces only become actionable when corresponding API capabilities exist.
- **Independent API/control plane:** authentication and session verification, consent/authorization records, input validation, quota and entitlement checks, job lifecycle, signed private-media operations, routing decisions, provider/worker lifecycle, usage/cost accounting, audit events and WebSocket authorization.
- **Signaling/session service:** session admission, short-lived credentials, offer/answer/ICE or equivalent negotiation, worker binding, health and termination. It must not expose provider credentials or allow arbitrary resource selection.
- **Python engine:** reusable, versioned processing package with explicit model/runtime requirements, input/output contracts, capability reporting and deterministic session lifecycle. Keep it independent from GUI, provider SDKs and deployment-specific control code. Do not change the current engine as part of this architecture approval.
- **GPU worker:** long-lived session process that loads approved models once, accepts bounded frame input, returns transformed output and telemetry, enforces timeouts/limits, and releases resources on session close or worker retirement.
- **Provider adapters:** map the platform's internal worker lifecycle to provider-specific APIs without leaking provider concepts into UI or engine contracts.

## Processing modes

1. **Local CPU:** proposed browser-local path; first investigate WebAssembly/WebGPU-capable runtimes and lightweight, legally usable models. The current Python desktop CPU path is not a browser implementation. If the required model cannot run safely and performantly in the browser, report unsupported rather than simulate processing.
2. **Cloud GPU:** remote persistent worker sessions for live media and provider-managed asynchronous workers for image/video jobs. Requires explicit user consent, secure media handling, capacity admission, usage/cost controls and operational ownership.
3. **Automatic hybrid:** policy chooses a supported local path or remote GPU path using capability, latency, resource availability, privacy preference and cost budget. It cannot silently upload local media; remote fallback requires prior informed opt-in and a visible transition.

See [hybrid processing](HYBRID_PROCESSING_ARCHITECTURE.md), [worker design](CLOUD_GPU_WORKER_ARCHITECTURE.md), and [WebRTC design](WEBRTC_STREAMING_ARCHITECTURE.md).

## Security and privacy invariants

- Require explicit authorization/consent for source likenesses and target media; distinguish camera preview from transformed output.
- Keep private media in private storage, use scoped short-lived access, minimize retention, and define deletion/revocation behavior before launch.
- Authenticate and authorize every API, job, WebSocket and signaling operation; scope records and media access to the authenticated principal/tenant.
- Keep provider credentials exclusively in backend/worker secret stores. Never return them to the browser, client logs, URLs or persisted frontend configuration.
- Apply file size/type/duration/resolution limits, malware/content policy controls as approved, rate limits, quotas and abuse response.
- Record operational metadata without face images, embeddings, raw frame data or unnecessary identifiers.
- Encrypt transport and storage; define key ownership, backup, incident response and data-region policies before production.

## Existing vs proposed vs unknown

| Area | Status |
|---|---|
| Local Python GUI, webcam capture, face detection/swap, image/video processing and pyvirtualcam adapter | Existing code/README capability; not an evaluation of production reliability or current hardware performance. |
| Browser web app, independent API, accounts, PostgreSQL, media service, WebSocket or WebRTC | Not found in inspected repository; proposed only. |
| Reusable headless service API for the current engine | Not verified; requires extraction/design and compatibility work, not assumed to exist. |
| Browser CPU/WASM face-swap implementation | Unverified investigation. |
| RunPod, Vast.ai, Modal, Replicate integration, pricing, GPU availability and measured latency | Not researched or integrated; provider-neutral evaluation required. |
| WhatsApp Desktop/Web/mobile compatibility | Not tested; see [assessment](WHATSAPP_COMPATIBILITY_ASSESSMENT.md). |
| Billing/subscriptions | UI requested; payment processor, entitlements, tax, refund and compliance approach undecided. |

## Decisions still required

Approve MVP modes and launch clients, identity/authentication approach, media retention/regions, consent and abuse policy, target model/license, latency/quality gates, provider evaluation criteria, cost ceiling, and whether remote GPU is in the first release. FastAPI plus PostgreSQL is a reasonable candidate for the independent control plane, not an implementation decision. Redis or another broker is optional and justified only if workload semantics require it. No vendor is selected.

## Related documents

- [Frontend/backend boundaries](FRONTEND_BACKEND_BOUNDARIES.md)
- [Hybrid processing](HYBRID_PROCESSING_ARCHITECTURE.md)
- [GPU provider abstraction](GPU_PROVIDER_ABSTRACTION.md)
- [Cloud GPU workers](CLOUD_GPU_WORKER_ARCHITECTURE.md)
- [WebRTC streaming](WEBRTC_STREAMING_ARCHITECTURE.md)
- [Implementation roadmap](VSWAP_WEB_IMPLEMENTATION_ROADMAP.md)
- [Legacy local pipeline assessment](REALTIME_VIDEO_PIPELINE.md)
