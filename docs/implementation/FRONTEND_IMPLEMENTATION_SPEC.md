# Frontend Implementation Specification

**Status:** Proposed route and interaction contract. No frontend exists in the inspected repository. Do not create components until explicit implementation approval.

## Recommended stack and standards

Next.js App Router, React, TypeScript, Tailwind CSS and shadcn/ui are recommended. Use reusable feature components and a typed API client. React Server Components for static/server-rendered shells; client components for camera, WebRTC, WebSocket and interactive controls. Data comes from the real API; never put persistent account/job/media data in localStorage. Provide truthful unavailable/loading/error/empty states. Mobile-first, accessible, minimum 44px controls, keyboard and screen-reader support, reduced motion, and 16px minimum input text. Layout and interaction should be verified at 414×765 dark mobile viewport before desktop.

Route protection and exact auth flow depend on the identity decision. No route implies backend capability. All processing actions are disabled or explained when `GET /v1/capabilities` says unsupported.

## Route specifications

| Route | Purpose / components | Interactions and APIs | Loading / error / empty behavior |
|---|---|---|---|
| `/dashboard` | Overview cards, recent jobs, capability/service status, usage summary. | `GET /capabilities`, `/jobs`, `/users/me/usage`. | Skeleton; API offline banner; no jobs/usage empty state; no fabricated metrics. |
| `/live-studio` | Camera preview, source selector, mode/quality, resolution profile, CPU/GPU capability, FPS/latency metrics, start/stop, diagnostics. | `GET /source-faces`, `/capabilities`; `POST /realtime/sessions`; WebSocket status/signaling; WebRTC media; stop endpoint. | Permission blocked/unavailable; separate preview and transformed outputs; waiting/start/stream/degraded/reconnect/failure; no “live swap” until transformed frames confirmed. |
| `/image-swap` | Source and target selection, consent, quality/profile options, job status/result. | Media upload endpoints; source faces; `POST /jobs`, status/events, result retrieval. | Upload progress/cancel; invalid media/consent/limit errors; empty library; no result until completed. |
| `/video-swap` | Target video selection, duration/size limits, consent, estimated budget if available, job timeline. | Same media/job APIs; WebSocket job events. | Validate codecs/duration; upload interruption recovery; queue/worker unavailable; output private. |
| `/faces` | Source-face list, add/delete, consent/authorization metadata. | Media and source-face endpoints. | No faces empty state; upload/validation; deletion pending; never show embeddings. |
| `/history` | Filtered jobs/sessions with state and result actions. | `GET /jobs`, session history API if approved. | Cursor pagination, no history state, backend outage, expired/removed media state. |
| `/gpu-resources` | User-safe capability, service availability, coarse wait and usage—not provider control plane. | `GET /capabilities`, usage. | Hide internal workers/providers/hosts; unavailable state; role-gated admin observability is separate. |
| `/settings` | Profile, privacy, remote-processing preference, retention options if supported, security/session actions. | User profile/settings/auth endpoints. | Save pending/error, revoked session, clear consent guidance; preferences do not imply per-job consent. |
| `/billing` | Plan, entitlement, usage, payment status and cancellation actions. | Usage/entitlement and selected payment integration endpoints. | No payment provider assumed; show not configured, pending webhook, failure; server remains entitlement authority. |

## Component boundaries

`AppShell`, responsive `Navigation`, capability service, `ApiClient`, error boundary, consent disclosure, `MediaPicker`, `SourceFaceSelector`, `CameraPermissionPanel`, `LocalCameraPreview`, `RealtimeSessionController`, `WebRtcTransport`, `JobStatusTimeline`, `MetricsPanel`, `PrivateMediaResult`, and form primitives are candidate units. Keep transport clients separate from page composition. Use semantic labels, visible focus, status announcements that avoid flooding screen readers, and touch-safe controls.

## Live Studio state model

`idle → requesting_permission → local_preview → creating_session → signaling → worker_starting → transformed_stream → degraded/reconnecting → stopping → ended`; failures are explicit branches. Local camera preview is not processing. Server-side ready + confirmed transformed output is required before showing successful processing. Stop must close tracks, peer connection, signaling and backend session. Mode selection must disclose local vs remote before start; switching to cloud requires explicit consent.

## Validation

Component/accessibility tests; API-client contract tests; browser E2E for no auth, permission denied, unsupported capability, successful actual service path, API outage, reconnect, stop, job completion/failure, mobile keyboard/viewport and screen reader labels. Use no mock processing success. Mock fixtures only in tests and visibly never treated as product data.
