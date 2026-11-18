# VSwap MVP Scope

**Status:** Proposal for explicit approval. It does not authorize source changes, paid services, or deployment. This scope narrows the wider live/WebRTC vision in the [Phase 2 blueprint](../MASTER_IMPLEMENTATION_BLUEPRINT.md) to a testable authenticated image-processing product.

## Product outcome

An authenticated user can upload an image they are authorized to use, explicitly consent to the selected processing path, submit a real asynchronous image-transformation job, observe its truthful status, view/download the private result, and inspect or manage their job history. An operator can see basic service/job health and sanitized errors without access to raw face media by default.

## MVP requirements (proposed baseline)

1. **Responsive frontend foundation:** mobile-first account shell, accessible navigation/forms, dashboard/image workflow/history/settings, loading/error/empty/offline/unavailable states. UI reflects backend capability discovery; it does not claim unavailable capabilities.
2. **Authentication and sessions:** real, server-validated authentication and session expiry/revocation; protected user routes and user-scoped data. No mock/client-only auth.
3. **Secure media upload:** private, scoped upload, decoded-image validation, checksum/size/dimension limits, consent record, retention, and verifiable deletion.
4. **Image-processing workflow:** one supported still-image transformation path (one target image per job; source-face selection only if the licensed engine supports it), server-side policy and capability checks, idempotent async job creation. No fake or placeholder successful output.
5. **Asynchronous status:** queued/running/completed/failed/cancelled/expired states with bounded progress and stable user-safe errors; UI status refetches authoritative server state.
6. **Result preview/download:** owner-only output, private storage, short-lived access, expiry/deletion behavior, and no permanent public URL.
7. **History/job management:** paginated user-scoped job history, status, retry rules, cancellation where possible, and deletion/expiry clarity.
8. **Basic administrative observability:** health/readiness, redacted structured logs, request/job IDs, aggregate queue/worker/error/latency/cost metrics, alerts and audit trails with role separation. Admin tools are not ordinary user UI.
9. **Abuse, safety, and privacy controls:** explicit per-job likeness/target authorization, remote-processing disclosure/consent, rate/size/concurrency/cost limits, content/abuse handling approved by policy, and data lifecycle enforcement.

### MVP acceptance envelope

- A real inference result is necessary to call the image workflow functional; local preview, a queued job, or a canned fixture is not success.
- Every API, job, asset, history item, and result is bound to the authenticated user/tenant and denial is tested.
- Invalid or over-limit media never enters inference. Consent and mode are rechecked server-side at job admission and before remote processing.
- Users see useful states for no capability, unavailable worker, rejected input, cancellation, timeout, and expired/deleted output.
- No production account, payment, storage, queue, or compute service is selected by this scope; approval is required for each.

## Deferred from MVP

- Video transformation (separate duration, codec, upload, cost and resource policy).
- Live webcam transformation, WebRTC signaling/media, persistent GPU sessions, TURN, latency SLOs, session affinity and reconnect semantics.
- WhatsApp Web/Desktop/mobile compatibility claims; system virtual cameras; Windows/macOS camera agents; Android/iOS native apps.
- Automatic local/cloud hybrid routing; silent failover is prohibited.
- Billing, subscriptions, payments, commercial entitlements, marketplace/social features, multi-user collaboration and teams unless product approval changes scope.
- Multi-agent functionality, voice/audio transformation, model training, reusable biometric embeddings, broad video history or public sharing.
- Production deployment, auto-scaling, vendor-specific GPU provisioning, and broad region expansion.

## Experimental, not an MVP commitment

- Browser-local CPU/WASM/WebGPU inference; desktop CPU support does not imply browser support.
- GPU provider comparison, cold-start/capacity tests, live-session prototype, and WhatsApp camera feasibility. These require explicit approval for any external data use or cost.
- Video, WebRTC, native-camera, and hybrid prototypes may only be labeled experimental, isolated, and non-production; they must not appear as working user capabilities.
- Any experiment using personal media requires approved consent, region, retention and security conditions first.

## External services and GPU boundary

| Need | MVP status | Notes / gate |
|---|---|---|
| Authentication | Required | Hosted identity provider vs self-hosted auth remains open; provider choice and data-processing terms require approval. |
| PostgreSQL | Required for durable user/job/consent state | Managed vs self-hosted and region unresolved; no database exists in this repo. |
| Private object storage | Required | Provider, region, key management, backups, egress and deletion policy unresolved. |
| Queue/outbox | Required async dispatch semantics | Begin with durable DB/outbox abstraction if acceptable; add a separate broker only with measured justification and approval. |
| Email / account recovery | Conditional | Needed if selected auth flow requires it; provider and delivery of personal data require approval. |
| Monitoring/alerting | Required before staging/public availability | Provider, data retention and spend unresolved; logs must be redacted. |
| GPU inference | Conditional infrastructure, not pre-approved | Processing needs a capable execution environment. A GPU is required only if the approved model/latency/throughput cannot meet the approved target on an acceptable CPU worker. If GPU is needed, provider, region, isolation and spending need approval before provisioning. |
| Model and weights | Absolutely required for real image processing | Current README cites research/non-commercial limitations for models. Rights review is a hard gate; a model that cannot lawfully support the intended deployment blocks M6 and user-facing processing. |
| Vercel | Frontend preview/hosting candidate only | Current project is configured as Python at root and failed deployment. FastAPI jobs/long-running workers/GPU/WebRTC should not be presumed suitable for Vercel Functions. |

## Non-goals and invariants

Do not modify or replace the existing desktop product as a side effect. Do not copy model weights, user media, credentials, or generated output into Git. Do not represent stub data as live account/job data. Do not expose raw media, embeddings, worker identifiers, provider addresses, or tokens in telemetry or browser responses. Do not enable remote processing as a side effect of a saved preference; consent is per operation. No paid infrastructure or production deployment without explicit permission.

## Source-of-truth links

- [Phase 3 roadmap](PHASE_3_ROADMAP.md)
- [Pending decisions](TECHNICAL_DECISIONS_PENDING.md)
- [Phase 2 master blueprint](../MASTER_IMPLEMENTATION_BLUEPRINT.md)
- [Testing and acceptance proposal](../TESTING_AND_ACCEPTANCE_PLAN.md)
- [Repository-specific web roadmap](../../VSWAP_WEB_IMPLEMENTATION_ROADMAP.md)
