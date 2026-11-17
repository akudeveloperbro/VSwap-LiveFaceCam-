# Versioned API Implementation Specification

**Status:** Proposed contract, not implemented endpoints. Independent FastAPI service is the leading candidate. Schemas below are illustrative and must be finalized with auth, tenancy, consent and retention decisions.

## Conventions

Base path `/v1`. JSON UTF-8. IDs are opaque UUID strings. Timestamps are RFC 3339 UTC. Authenticate all user endpoints; authorize each object by user/tenant. Cookie sessions require CSRF protections; alternative bearer transport requires short lifetime and protected storage. Enforce request size, schema, rate, concurrency and idempotency limits server-side. Use stable error shape:

```json
{"error":{"code":"MEDIA_TOO_LARGE","message":"The file exceeds the allowed limit.","retryable":false,"request_id":"opaque-id"}}
```

Never expose stack traces, secrets, worker addresses, provider tokens or cross-user object existence.

## Authentication / profile

Auth implementation/provider remains unresolved. If owned by backend: `POST /auth/register`, `POST /auth/login`, `POST /auth/logout`, `GET /users/me`, `PATCH /users/me`. Credential response contains no password/hash; session cookie is Secure, HttpOnly, appropriate SameSite, rotated and revocable. Login/register response and status must resist account enumeration. Do not expose these endpoints until auth/session architecture is selected.

## Capabilities and settings

- `GET /capabilities`: actual server-enabled operations, limits, model/runtime capability IDs, supported input formats and service availability. Distinguish `available`, `unavailable`, and `not_configured`; never claim processing based on UI presence.
- `GET/PATCH /users/me/settings`: allowlisted settings only. Remote-processing preference is not consent for a specific media/session.
- `GET /users/me/usage`: backend-authoritative aggregate usage/limits.

## Media

- `POST /media/uploads` with `{kind, filename, declared_mime_type, byte_size, sha256}` returns `{asset_id, upload_method, upload_url, expires_at, constraints}`. URL is short-lived, private and scoped to one object; never accept arbitrary object URLs.
- `POST /media/{asset_id}/complete` validates uploaded bytes, checksum, actual media decode, malware/policy checks and size/duration/resolution, then returns `{asset_id,status,metadata}`.
- `GET /media?kind=&cursor=` returns only owner-scoped metadata, not a permanent object URL.
- `DELETE /media/{asset_id}` begins idempotent deletion and returns `202 {asset_id,status:"deletion_pending"}`; completion state is observable and verified after object deletion.
- Source faces: `POST /source-faces` references an authorized asset and consent record; `GET /source-faces`, `DELETE /source-faces/{id}`. Do not store embeddings by default.

## Batch processing

- `POST /jobs` requires `Idempotency-Key`; body: `{kind:"image"|"video", input_asset_id, source_face_id, options, consent:{policy_version,remote_processing}}`. Server rechecks ownership, consent, limits, entitlement and mode. Returns `202 {job_id,status:"queued",created_at}`.
- `GET /jobs/{job_id}` returns `{job_id,status,progress,created_at,started_at,finished_at,error?,output_asset_id?}` owner-scoped; output asset is not a public URL.
- `POST /jobs/{job_id}/cancel` idempotently requests cancellation; returns current status.
- `GET /jobs/{job_id}/result` returns a short-lived authorized download operation only for completed owner-owned jobs.

No per-frame endpoints. Avoid accepting arbitrary processing provider selection from clients; expose product-level `mode`/quality profiles validated against capabilities.

## Processing mode discovery and selection

`GET /processing/modes` may return only currently configured supported modes and their capability/constraint metadata. The client requests a product mode such as `local`, `cloud`, or `automatic` in `POST /jobs` or `POST /realtime/sessions`; the backend validates the choice against consent, entitlements, capability and cost policy. It does not accept a provider name or arbitrary worker ID from a user. `automatic` must not fall back to remote processing without explicit remote consent.

## Realtime sessions

- `POST /realtime/sessions` body `{source_face_id, mode, profile, consent:{policy_version,remote_processing}}`; returns `201 {session_id,status:"starting",expires_at,signaling:{url,credential},media:{transport:"webrtc"}}`. Credentials short-lived, session-bound and never persisted in URL/logs.
- Signaling WebSocket: authenticated messages `offer`, `answer`, `ice_candidate`, `restart`, `close`; enforce schema, sequence, size, rate, expiry and session binding. Worker/provider identifiers are internal.
- Events: `session.status`, `session.metrics`, `session.error`; include `event_id`, `sequence`, `occurred_at`, `session_id`, sanitized payload. Resume/replay behavior and retention must be specified.
- `POST /realtime/sessions/{id}/stop` requests idempotent close; server invalidates credentials and tears down worker. `GET /realtime/sessions/{id}` returns status/aggregate metrics.

WebRTC carries media only; signaling may use WebSocket. Authenticated control events do not contain frames, embeddings, credentials or secret endpoints.

## Infrastructure (admin-only)

Provider/worker resource status belongs to internal admin/operations API, not public user API. Public `GET /capabilities` may expose coarse availability/estimated wait band only. `GET /admin/providers`, `/admin/workers`, `/admin/resources` requires a separate privileged role, audit trail and response redaction. No API lets normal users provision arbitrary GPUs.

## Status/error semantics and tests

Common statuses: `queued`, `starting`, `running`, `degraded`, `completed`, `failed`, `cancelled`, `expired`, `deletion_pending`. HTTP: 200 read/update, 201 resource created, 202 accepted async/cancel/delete, 400 schema, 401 unauthenticated, 403 unauthorized/consent, 404 owner-scoped absent, 409 idempotency/state conflict, 413 media size, 422 decoded validation, 429 quotas/rate limits, 503 unavailable. Exact mapping requires contract review. Tests must cover schema, auth/ownership, idempotency, replay, rate limits, malformed inputs, quotas, revocation, deletion and OpenAPI compatibility.
