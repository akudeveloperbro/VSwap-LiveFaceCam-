# Backend Implementation Specification

**Status:** Proposed, not implemented. Python + FastAPI is the leading recommendation; PostgreSQL recommended for durable relational state. Confirm before coding.

## Recommendation and trade-offs

FastAPI fits the existing Python engine ecosystem and provides typed request/response validation and OpenAPI generation. Keep API orchestration separate from inference so slow GPU work cannot block request handlers. PostgreSQL fits relational ownership, job/session state, quotas, audit and usage reconciliation. Candidate persistence is SQLAlchemy 2.x with Alembic; explicit SQL is also viable for a smaller service, but choose one approach and test migrations. Redis is optional: use for ephemeral locks, short-lived presence or broker functionality only if delivery semantics/scale require it; do not make it the durable source of job/session truth. For durable async delivery, compare a managed queue/outbox design against Redis based on retries, ordering, delivery guarantees and operational burden. Private S3-compatible object storage is required for remote media; exact vendor/region is unresolved.

## Service modules

`api/v1` routing and schemas; `auth` adapter; `domain` jobs/sessions/media/consent/usage; `repositories` scoped database access; `services` policy and lifecycle; `storage` private media adapter; `queue`/outbox; `orchestration` mode/capacity/cost decisions; `providers` adapter interfaces; `realtime` signaling/session admission; `workers` control protocol; `observability` redacted metrics/audit; `migrations` Alembic. No worker-specific code in browser API routes beyond control messages.

## Reliability patterns

Use transactionally written outbox for state changes requiring reliable event/queue publication; idempotency keys for job creation and worker allocation; bounded retries with jitter only for transient conditions; leases/TTL and orphan reconciliation; cancellation tokens; explicit states and optimistic concurrency/version; database constraints for invariants; timeouts at every network boundary. Never hold DB transactions across inference or WebRTC lifetime. WebSocket events are not the source of truth; clients can refetch current state.

## Security controls

Auth/session mechanism undecided; select a maintained solution and define cookie/token lifecycle. Centralize user/tenant scoping in repository/service access; admin access separate and audited. Validate media after decoding and scan according to approved threat model. Use short-lived scoped object access, private buckets, encryption and deletion verification. Provider credentials in managed secrets only. Rate-limit login, upload, jobs, sessions and signaling. Enforce quota/resource/cost limits server-side. Use CORS allowlist, CSRF protection when cookie-based, secure headers, input schema validation, output redaction, request IDs and abuse monitoring. Treat face images and derived data as sensitive; do not store embeddings by default.

## Deployment/service boundaries

Independent API/control-plane deployment; job scheduler/worker control service; WebSocket signaling service; WebRTC/media gateway; GPU worker deployment. These may begin as a modular monolith/control plane plus separate worker process, but GPU inference and long-running media should not run in a serverless request lifecycle. Database/storage/network regions, scaling model, secrets, backups, alerts and RTO/RPO need approval. Provide liveness/readiness separately; readiness checks dependencies without leaking credentials.

## Acceptance gates

OpenAPI diff review; migrations tested on empty and prior schema; auth/ownership/tenant isolation tests; idempotent retries; outbox replay; worker lease recovery; privacy deletion checks; rate and quota tests; structured error and log redaction tests; load test at approved concurrency; backup/restore and incident runbooks. No public production API before threat model and operational owner approval.
