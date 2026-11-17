# Frontend / Backend Boundaries

**Status:** Proposed contract boundaries; no web/API implementation is present in the inspected repository. Updated 2026-11-18.

## Ownership

| Concern | Web frontend (v0 / Next.js) | Independent backend |
|---|---|---|
| Presentation and UX | Owns responsive dashboard, controls, accessible state, camera permission UX and client-side input feedback. | Supplies capability, policy and job/session state. |
| Identity | Uses the selected authentication flow and sends its session credential through the supported secure mechanism. | Owns authoritative authentication, session validation, revocation, authorization and tenant/user scoping. |
| Media | Acquires browser camera only after user action; previews locally; uploads only after explicit consent. | Validates and stores media privately; issues scoped, expiring upload/download authorization; enforces retention and deletion. |
| Processing | Requests a declared mode/capability; renders server-reported states and output. | Selects resources, schedules, limits, runs jobs and accounts for usage/cost. |
| Realtime | WebSocket control/progress and WebRTC media; handles reconnects and lifecycle visibly. | Authenticates signaling/events, admits sessions, binds workers and terminates sessions. |
| Billing | Displays backend-authoritative plan, entitlement and billing status. | Owns entitlement enforcement and verified payment-provider events. |

Next.js server routes may act as a narrow authenticated BFF only if needed, but must not become the inference runtime, persistent media store, provider manager or source of truth for jobs. Prefer a versioned independent API. No GPU/provider secret reaches the browser.

## Proposed API families (not deployed contracts)

- `GET /v1/capabilities` — actual enabled modes, limits, supported formats and current service availability; include capability/version metadata.
- `POST /v1/media/uploads` and completion endpoint — create a constrained private upload operation; never accept arbitrary URLs or public-by-default objects.
- `POST /v1/jobs` / `GET /v1/jobs/{job_id}` / cancel endpoint — asynchronous image/video work, idempotency key, validated media IDs, explicit consent and processing options.
- `POST /v1/realtime/sessions` / close endpoint — authorize a session and return short-lived, scoped signaling details; session policy includes expiration and resource budget.
- Authenticated WebSocket event stream — job/session state, progress and coarse metrics. Define sequence IDs, replay/reconnect behavior, heartbeat, event schema and backpressure.
- `GET/PATCH /v1/account/settings`, usage and billing endpoints — only after account/billing decisions and authorization policy are approved.

These are endpoint families for discussion, not claims about existing routes. Versioning, schemas, auth mechanism, pagination, error format, idempotency retention and rate limits must be agreed before implementation.

## Trust and validation rules

- Treat all browser fields, filenames, MIME types, durations, codec metadata, job IDs, provider preferences and WebRTC messages as untrusted.
- Validate input on the server; enforce actual decoded media constraints, not just declared content type or client-side checks.
- Scope every database lookup and object-storage operation to the authenticated user/tenant and authorized job/session.
- Use short-lived scoped credentials for upload and media retrieval. Avoid long-lived bearer tokens in URLs, browser storage or logs.
- Apply CSRF protections where cookie auth is used, secure cookie attributes, origin checks, rate limiting and session revocation.
- Make operations idempotent where retries may duplicate uploads, job creation, billing events or worker allocation.
- Use generic client errors; keep stack traces, provider identifiers, internal hosts and credentials server-side.
- Publish only metrics needed for UX. Do not stream raw frames or identity embeddings through control events.

## Data model candidates (not schema)

`User`, `Session`, `ConsentRecord`, `MediaAsset`, `ProcessingJob`, `RealtimeSession`, `WorkerLease`, `UsageEvent`, `ProviderHealthSnapshot`, `BillingEntitlement`, and `AuditEvent`. Ownership, retention, encryption, deletion, status transitions and immutable billing/audit fields require explicit schema design. Do not create tables until identity, tenancy and retention requirements are approved.

## Error/state contract

Every user-visible operation should distinguish `queued`, `starting`, `running`, `completed`, `failed`, `cancelled`, and `expired` where applicable. Include stable machine-readable error codes, retryability and correlation IDs. A camera preview state must not be conflated with processing readiness or a successful transformed frame. Backend unavailability, queue saturation, no eligible worker, quota denial, provider failure, permission denial, expired upload and lost signaling each need an explicit non-success state and recovery affordance.

## Acceptance gates

Before implementation: approve auth/session design, user/tenant model, API versioning, consent model, media restrictions/retention, event delivery semantics, quota behavior and ownership of operational alerts. Before release: contract tests, auth/ownership denial tests, malformed/oversized media tests, retry/idempotency tests, deletion verification and browser states for loading/error/empty/offline.
