# Phase 3 — Prioritized Implementation Backlog

**Status:** Proposed tasks, not authorized work. All source paths are targets only; inspect the repository before implementation. The repository currently contains a desktop Python app, and no frontend/backend services. No item below changes that fact.

Task size target: one focused coding-agent session with a narrow reviewable diff. Split a task if it spans more than one milestone, needs unresolved policy choices, or cannot be independently tested.

## P0 — Blocking approvals and feasibility

### P0-01 — Approve MVP and launch claims
- **Description:** Approve image-only first inference slice, user groups, supported image types, explicit exclusions, and claims policy.
- **Prerequisites:** Read [MVP scope](MVP_SCOPE.md) and [Phase 2 blueprint](../MASTER_IMPLEMENTATION_BLUEPRINT.md).
- **Files:** Decision entry in `../ARCHITECTURE_DECISION_REGISTER.md` only after owner approval; no source files.
- **Definition of done:** Named approver records scope and excludes video/live/WhatsApp/native unless separately authorized.
- **Tests:** Review scope against route/API docs and ensure every advertised capability has an acceptance condition.

### P0-02 — Clear model, weights, and code rights
- **Description:** Determine whether each candidate model/weight, engine dependency, and service deployment may be used for the intended hosted product and data flow; select only a legally cleared alternative.
- **Prerequisites:** Rights holder/legal review; intended commercial/non-commercial operation stated.
- **Files:** Approved license inventory in a reviewed documentation location; no model weights committed.
- **Definition of done:** Written rights determination covers use, hosting/inference, redistribution/download, modifications, output, territories, and notices; candidate digest/source is recorded.
- **Tests:** Automated/checklist validation that approved model artifacts match recorded hashes; legal sign-off is not replaced by software tests.

### P0-03 — Approve consent, abuse, retention, and region policy
- **Description:** Define who may be depicted, proof/attestation required, revocation/reporting, disclosure, raw input/output/temp retention, backups, deletion verification, processing regions and incident handling.
- **Prerequisites:** Product, privacy, security and legal decision owners.
- **Files:** Approved policy/decision docs; no application source.
- **Definition of done:** Specific per-operation consent and remote-processing disclosure are unambiguous; data lifecycle and response owners documented.
- **Tests:** Trace policy to database, API, UI and worker scenarios; negative cases (revoked/expired/unconsented) have defined expected outcomes.

### P0-04 — Approve cost/resource and hosting guardrails
- **Description:** Set global and per-user budgets, quotas, concurrency, timeout, egress/storage limits, provider approval process, and deployment boundary.
- **Prerequisites:** Product owner and infrastructure/finance approver.
- **Files:** Decision register and future environment/infra docs; no paid resource creation.
- **Definition of done:** Written numeric caps and shutdown/alert behavior exist; Vercel root/frontend and API/worker hosting decision is assigned.
- **Tests:** Review cap calculations and fail-closed paths; no provisioning before approval.

### P0-05 — Select identity and tenancy approach
- **Description:** Choose hosted identity or self-managed auth, session transport, account recovery, user/tenant ownership model, and deletion behavior.
- **Prerequisites:** P0-03, privacy/security review.
- **Files:** Decision register; later `frontend/src/lib/auth/**`, `backend/app/auth/**`, migration paths only after approval.
- **Definition of done:** One authoritative identity/session model, verified server-side, with account lifecycle and tenant boundary documented.
- **Tests:** Threat-model session expiry/revocation, CSRF if cookie-based, IDOR and enumeration cases.

## P1 — MVP critical (execute in milestone order after P0 approval)

### P1-01 — Establish isolated frontend workspace
- **Prerequisites:** M0 exit; frontend framework, package manager, visual/a11y baseline approved.
- **Files:** `frontend/package.json`, lockfile, `frontend/src/app/**`, `frontend/src/components/**`, frontend-specific config.
- **Definition of done:** Clean install and dev server work without changing root desktop setup; baseline route shell is mobile-first.
- **Tests:** Lint/typecheck/unit/browser smoke; 414×765 mobile and desktop screenshots; no fake success.

### P1-02 — Implement capability-driven screens and states
- **Prerequisites:** P1-01; versioned capability contract.
- **Files:** `frontend/src/app/**`, `frontend/src/features/**`, `frontend/src/lib/api/**`.
- **Definition of done:** Dashboard, image entry, history and settings show actual available/unavailable/error/empty/loading state.
- **Tests:** Component/contract/E2E state matrix; accessibility and mobile viewport checks.

### P1-03 — Create isolated FastAPI service baseline
- **Prerequisites:** M0 exit, API contract approval, runtime/hosting decision.
- **Files:** `backend/pyproject.toml`, backend lock, `backend/app/main.py`, `backend/app/api/v1/**`, config/observability/tests.
- **Definition of done:** Versioned OpenAPI, liveness/readiness, typed errors, redacted request-ID logs; no inference in request handler.
- **Tests:** Unit/API contract/config/CORS/log-redaction and health checks from clean install.

### P1-04 — Add database migrations and scoped repositories
- **Prerequisites:** P0-05, PostgreSQL/region decision, P1-03.
- **Files:** `backend/app/db/**`, `backend/app/repositories/**`, `backend/migrations/**`, database tests.
- **Definition of done:** Minimum user/consent/job/media metadata schema and migration workflow; every user query scoped by authenticated principal/tenant.
- **Tests:** Empty DB migration, migration rollback/forward recovery, repository owner isolation, constraints and backup/restore procedure.

### P1-05 — Implement real auth/session and account boundaries
- **Prerequisites:** P0-05 and P1-04.
- **Files:** `backend/app/auth/**`, `frontend/src/features/auth/**`, `frontend/src/lib/auth/**`, integration tests.
- **Definition of done:** Approved provider/session integrated; protected routes and server authorization work; expiry/revocation is enforced; no mock auth.
- **Tests:** Login/session lifecycle, CSRF as applicable, expiry/revoke, enumeration resistance, unauthenticated and cross-user denial.

### P1-06 — Implement private upload and policy validation
- **Prerequisites:** P0-02/03/04, P1-04; object storage/region and limits approved.
- **Files:** `backend/app/media/**`, `backend/app/storage/**`, media migrations/tests.
- **Definition of done:** Scoped upload, decoded-image validation, size/dimension checks, consent association, private storage, verified deletion state.
- **Tests:** Corrupt/polyglot/oversized media, MIME mismatch, signature expiry, unauthorized reads, deletion retries, object-store outage.

### P1-07 — Implement durable image job lifecycle and dispatch
- **Prerequisites:** P1-06; queue/outbox, retry, cancellation, concurrency and cost controls approved.
- **Files:** `backend/app/jobs/**`, `backend/app/queue/**`, `backend/app/outbox/**`, migration and job tests.
- **Definition of done:** Idempotent owner-scoped asynchronous status; bounded retries; cancellation and terminal failures; committed jobs not lost on transient queue failure.
- **Tests:** State machine, duplicate event/outbox replay, queue outage/saturation, lease/retry/cancel races, cross-user status denial.

### P1-08 — Integrate approved licensed image engine
- **Prerequisites:** P0-02/03/04, P1-07; approved model/runtime/provider and worker security boundary.
- **Files:** `ai-engine/**` or approved isolated engine path; `backend/app/engine/**`; `gpu-workers/**`; integration tests. Do not edit existing `modules/` without a separate extraction review.
- **Definition of done:** Real image transformation in isolated worker, no Qt/camera needed, pinned approved model/runtime, bounded resource use and deterministic cleanup.
- **Tests:** Real authorized synthetic fixture, output baseline, missing/corrupt model, timeout/OOM/cancel/crash, cleanup, license/hash gate.

### P1-09 — Complete results, download and history UI
- **Prerequisites:** P1-02, P1-05–08; stable API and retention contracts.
- **Files:** `frontend/src/features/image/**`, `frontend/src/features/history/**`, backend result/history routes and E2E tests.
- **Definition of done:** Real job status, private owner-only result, expiry-aware download, cancel/history and truthful failures all work.
- **Tests:** Authenticated E2E success/failure/expired/denied/upload-interrupted/API-offline, mobile accessibility.

### P1-10 — Add MVP security, privacy, reliability and ops gates
- **Prerequisites:** P1-01–09; security/privacy review owners.
- **Files:** frontend/backend tests, `tests/contract/**`, `tests/e2e/**`, CI workflows after approval, runbooks.
- **Definition of done:** No critical/high issue; required auth/media/deletion/abuse tests pass; redacted metrics and useful alerts exist.
- **Tests:** Security/dependency audit, IDOR/CSRF/rate/quota, load/fault, privacy deletion, backup restore, all supported mobile browsers.

## P2 — Important after MVP evidence

### P2-01 — Video jobs
- **Prerequisites:** P1 MVP complete; separate codec/duration/size/retention/cost and model policy approval.
- **Files:** `backend/app/jobs/video/**`, `frontend/src/features/video/**`, worker and media tests.
- **Definition of done:** Bounded video input/output lifecycle and accurate progress/cost/failure UI.
- **Tests:** Codec/duration/resource limits, interrupted upload, worker crash, cleanup and cost ceilings.

### P2-02 — Operational dashboards and restore drills
- **Prerequisites:** P1-10; monitoring provider and retention approved.
- **Files:** `backend/app/observability/**`, approved monitoring config, runbooks.
- **Definition of done:** Aggregate queue/latency/failure/cost/worker health and alert tests; no raw media or credentials in telemetry.
- **Tests:** Alert simulation, log/metric redaction, backup restore and incident exercise.

### P2-03 — Browser-local inference feasibility
- **Prerequisites:** Separate experiment approval and compatible model license.
- **Files:** Isolated `experiments/browser-inference/**`, device test report.
- **Definition of done:** Measured feasibility report; unsupported is an acceptable result; no user-facing capability claim without pass.
- **Tests:** Supported browser/device matrix, memory/thermal/quality/performance and download-size measurements.

## P3 — Deferred/experimental

- **P3-01:** WebRTC live video and GPU session affinity — prerequisites: M6 success, topology/privacy/cost approval; proposed `frontend/src/features/live/**`, signaling/media/worker paths; DoD requires actual transformed remote output and approved SLO; test sustained/network/reconnect/cleanup.
- **P3-02:** WhatsApp Desktop/Web/mobile compatibility — prerequisites: public OS/client API and distribution/legal review; native/compatibility test paths; DoD is exact named build matrix evidence, otherwise unsupported; tests use authorized subjects and remote participants.
- **P3-03:** Native virtual camera, mobile apps, hybrid auto-routing, billing/subscriptions, multi-agent/voice features — each requires separate product/architecture/cost/security approval and separate backlog decomposition.

## Desktop maintenance (separate track)

`ROADMAP.md` contains open desktop defects and enhancements. Do not fold these into VSwap web MVP unless an approved issue is shown to block the selected engine integration. Any required engine extraction must preserve desktop behavior and legacy tests; isolate it as a new explicitly approved task.
