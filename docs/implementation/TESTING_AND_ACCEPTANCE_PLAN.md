# Testing and Acceptance Plan

**Status:** Proposed test strategy; no test infrastructure for the proposed services is verified. No source implementation performed.

## Test layers

| Layer | Test strategy | Release gate |
|---|---|---|
| Unit | Frontend state/components; API policy/validation; database repositories; engine stages/adapters; worker lifecycle; provider normalization. Use synthetic licensed test frames only. | Deterministic, isolated tests; failure/edge branches covered. |
| Contract/API | OpenAPI schema, client generation, status/error codes, idempotency, auth, ownership, pagination, WebSocket event versions. | Breaking contract changes versioned and reviewed. |
| Integration | PostgreSQL migrations, object-storage private ACL/signed expiry, queue/outbox replay, worker control, secret boundaries. | Clean install/migrate; rollback/restore documented; cross-user denial passes. |
| Inference | Model/runtime digest, quality regression, detector/tracker/swap/composite, CPU/GPU behavior and memory. | Model/license approved; reproducible fixture and hardware/runtime recorded. |
| GPU worker/provider | Provision/health/terminate idempotency, no capacity, boot timeout, OOM, provider outage, orphan lease, cost reconciliation. | No leaked provider credentials; bounded recovery and approved spend cap. |
| WebRTC | Browser/device matrix, ICE/TURN, codec negotiation, signaling auth/replay, jitter/loss/bandwidth, reconnect/stop, transformed output. | Actual transformed video returns; metrics available and cleanup verified. |
| Camera compatibility | OS virtual-device enumeration, client selection, transformed frames to remote participant, sustained session, permission/device contention. | Exact OS/client/build matrix row passes; no general claim from a single row. |
| Load/performance | API/job concurrency, queue saturation, live streams, DB/storage/provider limits; compare cost and tail latency. | Capacity, queue, cost and p95 targets approved and met. |
| Security/privacy | Threat model, dependency/image scan, auth/tenant IDOR, CSRF/CORS, upload parser abuse, rate/quota, retention/deletion, log redaction, incident response. | No critical/high unresolved findings; privacy/deletion and abuse sign-off. |
| End-to-end | Browser happy path plus no auth, empty, offline, permission denied, capability unavailable, worker/provider failure, cancel, delete. | No fake success; user sees correct state across failures and mobile viewport. |

## Proposed measurable criteria (approval required)

- **Live baseline:** 1280×720 at 30 FPS for 10 minutes on a documented reference device; p95 capture-to-return age ≤100 ms; report actual output FPS, p50/p95 age, jitter, drop rate and memory trend. Repeat 3 runs; every run recorded. This is a proposed gate, not evidence.
- **Future optimization:** investigate 1920×1080 at 60 FPS only after 720p30 stable; quality and thermal constraints remain equal acceptance criteria.
- **Queue:** bounded, no monotonically growing frame age under overload; intentional drop count observable; user-visible degraded status if target missed.
- **API:** 100% of user object access paths have owner/tenant authorization tests; duplicate Idempotency-Key does not create duplicate job/charge; quota limits enforced server-side.
- **Media:** invalid or oversized decoded asset rejected; private object not accessible anonymously; deletion completion means object and transient copies removed according to policy.
- **Recovery:** worker/provider/session failure converges to terminal/recoverable state within an approved timeout; no orphan worker past lease cleanup SLO.
- **Compatibility:** no WhatsApp claim until transformed output observed in a real authorized call on exact named OS/client and a sustained test passes.

## WhatsApp and camera compatibility matrix

Compatibility is an evidence claim per exact OS/client version, not a general platform promise. Current app has a `pyvirtualcam` wrapper, but no WhatsApp call has been tested in this repository.

| Target | Candidate path to investigate | Required validation / known limitation |
|---|---|---|
| Windows + WhatsApp Desktop | Existing desktop engine plus a supported Windows virtual-camera output (for example, an installed provider/OBS Virtual Camera if enumerated). | Confirm app camera picker accepts the device, permissions/driver installation and reboots, then verify transformed frames at a remote participant during a sustained call. Record exact WhatsApp/Windows/driver versions. |
| Windows + WhatsApp Web | Windows virtual camera selected through supported browser camera permissions. | VSwap web page cannot substitute a camera track into another origin; use OS device only. Test exact browser and WhatsApp Web release. |
| macOS + WhatsApp Desktop | Existing Python virtual-camera wrapper or another documented/signable macOS virtual-camera component, if accepted by current client. | Validate installation/signing/OS privacy permissions, device enumeration and remote call output. Do not remove signatures, inject code or bypass client policy. |
| macOS + WhatsApp Web | OS virtual camera selected in the browser, with normal browser permission. | Exact browser/client support and permission behavior; a VSwap WebRTC stream alone is not a system camera. |
| Android | No ordinary third-party virtual-camera path is assumed. Investigate only public APIs/approved distribution mechanisms. | Do not root/hook the device or use privileged/system-only virtual-device APIs. If no supported device path exists, mark unsupported. |
| iOS | No ordinary third-party system-wide virtual camera is assumed. Investigate documented app extensions/OS capabilities only if actually available to third-party apps. | Do not jailbreak, hook or inject. Screen broadcast is not a camera device. Validate camera picker and remote call only on a supported path. |

For every supported row, capture a consenting synthetic/authorized subject; verify transformed image is visible remotely, then test camera denial/contention/disconnect, network/worker failure, stop/cleanup and thermal/memory stability. A local preview, successful enumeration, marketing claim, or one desktop platform is not proof for other clients.

## Data/evidence protocol

Record build/commit, OS/browser, device CPU/GPU/driver, engine/model digest/license, camera, mode, region/provider (if any), requested/actual resolution, network conditions, test duration and repetitions. Use consented synthetic/authorized subjects, avoid storing raw recordings, and retain only aggregate results needed for review. Label unmeasured values unavailable rather than zero or estimated.
