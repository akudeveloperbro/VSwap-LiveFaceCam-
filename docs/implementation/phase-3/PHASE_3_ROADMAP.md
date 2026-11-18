# Phase 3 — Implementation Roadmap

**Status:** Proposed execution plan only. This document does not authorize implementation, deployment, or spending. No application source has been changed. All target paths below are proposed and must be checked immediately before each approved task.

## Goal and repository audit

Deliver a controlled, consent-led image-processing MVP while preserving the existing LiveFaceCam desktop application. The current repository is not a blank web scaffold:

- Current branch: `docs/implementation-blueprint`, at `daa2c100a2d6d7f4e59c5049ea71722ff2080282`; clean before this plan. `main` and `origin/main` are both at `a901c5d36de9107a8726330b89897e9e956626d3`. The current branch adds 11 implementation specifications relative to `main`.
- PR #1 is merged. PR #2, “Add VSwap implementation blueprint,” is open, not draft, and not merged. Its head is the current commit; merge state is `UNSTABLE`.
- Existing application is a Python/PySide6 desktop GUI with camera capture, local image/video processing, ONNX Runtime provider selection, model downloading, and a `pyvirtualcam` adapter. `run.py`, `modules/`, assets, installers, locales, and six Python tests are existing working areas to preserve. A reusable headless engine is not established.
- Root dependencies in `requirements.txt` are a large desktop/inference stack (including PySide6, OpenCV, InsightFace, ONNX Runtime variants and TensorFlow). `pyproject.toml` currently configures Ruff only. No `package.json`, Node lockfile, frontend, FastAPI backend, Docker/Compose, database migrations, storage/queue adapter, or `vercel.json` was found. No dependency lockfile was found.
- CI currently has two GitHub Actions workflows: Python `compileall` and Ruff. The six existing tests are not run by either workflow. Do not treat their existence as evidence they currently pass.
- Vercel project `vswap-livefacecam` is configured with Root Directory `.`, Framework Preset `Python`, Node.js Version `26.x`, no build command, and install command `pip install -r requirements.txt`. The failed PR check is explained by its build log: `[PYTHON_ENTRYPOINT_NOT_FOUND]`; it expects a Python web entrypoint, while the repository is a desktop app and has no Vercel function entrypoint. The PR’s Vercel Preview Comments check passed. No Vercel/project setting or deployment was changed.
- `ROADMAP.md` contains open desktop bug reports B1–B11 (B1–B9 marked open; B10–B11 are not marked done) and feature/performance work; F10 is marked done. They are a separate desktop maintenance backlog, not MVP work. Do not combine or “clean up” those issues during web milestones.
- Material architecture constraint: README documents model weights with non-commercial/research limitations and states the project remains free/non-commercial. This does not prove a web deployment has legal rights to process media with those weights. A legally cleared model or written rights determination is an M6 blocker. Resolve model, weight, dependency, and AGPL obligations before remote deployment.

The checked-in [master blueprint](../MASTER_IMPLEMENTATION_BLUEPRINT.md), [MVP/architecture roadmap](../../VSWAP_WEB_IMPLEMENTATION_ROADMAP.md), [API specification](../API_IMPLEMENTATION_SPEC.md), [monorepo proposal](../MONOREPO_STRUCTURE.md), and [decision register](../ARCHITECTURE_DECISION_REGISTER.md) are the Phase 2 sources. Where their wider web/live ambitions exceed this approved MVP proposal, the narrower scope below controls until product approval.

## Dependency map

```text
M0 repository/environment audit and approvals
 └─ M1 frontend shell + honest capability states
     └─ M2 FastAPI baseline + versioned contracts
         └─ M3 PostgreSQL + identity/session/ownership
             └─ M4 private media upload/validation/deletion
                 └─ M5 durable image-job lifecycle + queue/outbox
                     └─ M6 licensed image engine integration
                         └─ M7 results, download, history and job controls
                             └─ M8 security, privacy, compatibility and load gates
                                 └─ M9 approved staging/deployment readiness
```

No milestone may skip a prerequisite by substituting fake processing success, permissive media access, mock production identity, or unapproved infrastructure. A contract can be documented and tested using explicit non-production fixtures, but product UI must label unavailable capability honestly.

## Milestones

Complexity is relative (S/M/L/XL), not a calendar estimate. “Files” are proposed paths; do not create them until the milestone is approved and each path is checked against the then-current tree.

### M0 — Repository & Development Environment

- **Objective:** Establish an accurate, reproducible baseline and decision/ownership gates without disturbing the existing desktop project.
- **Files:** Planning docs under `docs/implementation/phase-3/` (this phase only); if later approved, isolated `frontend/` and `backend/` manifests. Do not overwrite root `requirements.txt`, `pyproject.toml`, `run.py`, or legacy tests.
- **Dependencies:** Product/engineering owner; clarification of legal/privacy and budget decision owners.
- **Tasks:** Record branch/PR/source state; map manifests, scripts, existing CI, deployment settings, and tests; reproduce desktop and test commands from checked-in instructions; record baseline outcomes; choose isolated package/lockfile conventions; identify CI gaps; decide how Vercel will target the future frontend and where API/worker run; preserve a rollback point.
- **Acceptance:** Inventory is evidence-backed; clean-checkout setup and existing checks have recorded results; no legacy behavior/files changed; unresolved decisions have owner/approval gates; deployment ownership and root strategy are documented.
- **Tests:** Run existing documented smoke/tests only after checking environment and dependency cost; record commands, versions, failures. No inference claim from a desktop UI launch.
- **Risks:** Heavy OS/GPU-specific dependencies; Vercel currently points to a desktop Python repository root; requirements and README support claims may be stale.
- **Complexity:** M.
- **Rollback:** Documentation-only revert. Never delete/reinitialize the repository or change Vercel settings as part of M0.

### M1 — Frontend Foundation

- **Objective:** Build a responsive, accessible web shell with real capability-gated states; no processing success until a real service exists.
- **Files:** Proposed `frontend/package.json`, lockfile, `frontend/src/app/**`, `frontend/src/components/**`, `frontend/src/lib/api/**`, `frontend/tests/**`, and frontend-specific configuration. No root package manifest unless separately approved.
- **Dependencies:** M0; approved visual/accessibility baseline; route and capability contract; Vercel root-directory choice. Authentication UI can be a protected placeholder until M3, not a fake auth system.
- **Tasks:** Create mobile-first shell/navigation; implement dashboard, image workflow entry, history, settings, and explicit unavailable states; establish design tokens and reusable controls; implement typed API boundary/capability states, loading/error/empty/offline patterns, and accessible status announcements; no stored personal data in localStorage.
- **Acceptance:** At 414×765 and desktop, routes are responsive; touch controls are at least 44px; text inputs are at least 16px; keyboard/focus/screen reader basics work; unsupported backend/capability is visibly unavailable; no fabricated users/jobs/metrics/results; no camera capture or live features enter MVP.
- **Tests:** Component/unit, accessibility checks, browser E2E for loading/error/empty/offline/unsupported states, mobile viewport and navigation. No successful processing scenario until M7 integrates a real backend.
- **Risks:** UI may imply unsupported capabilities; Vercel currently builds the root as Python; introducing frontend deps into root could destabilize desktop setup.
- **Complexity:** M.
- **Rollback:** Revert only the frontend feature branch/commit; keep root desktop files and user data untouched.

### M2 — Backend Foundation

- **Objective:** Establish an independent, versioned FastAPI control-plane service with safe defaults and no inference in request handlers.
- **Files:** Proposed `backend/pyproject.toml` and lockfile, `backend/app/main.py`, `backend/app/config.py`, `backend/app/api/v1/**`, `backend/app/observability/**`, `backend/tests/**`; separate service/development instructions only after approval.
- **Dependencies:** M0; API owner; approved frontend contract; hosting/runtime direction. May proceed before DB-backed routes, but not production media endpoints.
- **Tasks:** Define versioned error/capability/health contracts; settings from environment with startup validation; liveness/readiness; request IDs and redacted structured logs; CORS allowlist and security headers; OpenAPI; local service command and isolated tests. Do not import desktop GUI or GPU dependencies into the API package.
- **Acceptance:** Clean install starts the service; health endpoints distinguish liveness/readiness; readiness does not leak secrets; OpenAPI is deterministic; handlers enforce schema and bounded request bodies; no media processing or unprotected user data endpoint exists.
- **Tests:** Unit, API contract/OpenAPI snapshot, config missing/invalid values, log redaction, CORS/security-header tests, health tests.
- **Risks:** Vercel Functions may not suit future persistent WebSockets/GPU workloads; API deployment should not be conflated with frontend deployment.
- **Complexity:** M.
- **Rollback:** Revert isolated backend commit; no shared database/schema or production traffic yet.

### M3 — Database & Authentication

- **Objective:** Add durable account-scoped records, secure authentication/session lifecycle, migrations, and authorization boundaries.
- **Files:** Proposed `backend/app/auth/**`, `backend/app/db/**`, `backend/app/repositories/**`, `backend/app/domain/users/**`, `backend/migrations/**`, `backend/tests/auth/**`, `backend/tests/authorization/**`.
- **Dependencies:** M2; approved auth provider/session design, database host/region, tenancy model, account deletion and retention policy. These are approval blockers.
- **Tasks:** Integrate selected auth provider or implement approved self-hosted auth; verify issuer/audience/session lifecycle server-side; add PostgreSQL migration strategy; create minimum users/settings/consent/audit/job metadata schema; scope every repository query by authenticated user/tenant; define logout/revocation/account deletion and migration rollback.
- **Acceptance:** No mock auth; unauthenticated access denied; cross-user object access denied; session expiry/revocation works; email/account responses do not enable enumeration; all migrations apply from clean DB and have reviewed rollback/forward recovery; no secrets logged or committed.
- **Tests:** Auth/session integration, CSRF when cookie-based, ownership/IDOR, tenant isolation, migration up/down on disposable DB, expiry/revocation, account deletion policy, logs redaction.
- **Risks:** Provider lock-in; biometric/media records intensify privacy exposure; destructive migrations and backups need a tested recovery plan.
- **Complexity:** L.
- **Rollback:** Roll back application release; use reviewed backward-compatible schema migration/expand-contract. Never drop user data to roll back.

### M4 — Private Media Infrastructure

- **Objective:** Upload, validate, privately store, authorize, retain, and verifiably delete user media.
- **Files:** Proposed `backend/app/media/**`, `backend/app/storage/**`, `backend/app/policies/media.py`, `backend/tests/media/**`, approved storage/deployment policy. Any bucket/IaC files require separate explicit approval.
- **Dependencies:** M3; object-store provider/region, encryption, retention/deletion, file limits, consent model, malware/content controls, budget.
- **Tasks:** Issue short-lived object-scoped upload authorization; verify upload completion against actual bytes/checksum; decode and enforce image-only MVP MIME/dimensions/size limits; bind assets to owner; prevent public URLs; implement deletion state/reconciliation; minimize logs and temporary copies.
- **Acceptance:** No unauthenticated or cross-user media access; declared MIME is not trusted; malformed, oversized, unsupported, and corrupt image inputs rejected; signed access expires and is object-scoped; deletion is not reported complete before storage confirmation; retention cleanup is observable.
- **Tests:** Upload boundary, MIME/decoder mismatch, polyglot/corrupt image, path/object-key tampering, signature expiry, cross-user access, deletion retry, object-store outage, temporary-copy cleanup.
- **Risks:** Raw images/face data are sensitive; storage egress, backup retention, and temporary worker copies can violate deletion promises.
- **Complexity:** L.
- **Rollback:** Disable new upload creation and preserve existing private objects; do not purge objects or revert retention data destructively. Roll forward cleanup safely.

### M5 — Processing Job System

- **Objective:** Build durable asynchronous image-job lifecycle, dispatch abstraction, worker leases, cancellation, retries, and truthful status.
- **Files:** Proposed `backend/app/jobs/**`, `backend/app/queue/**`, `backend/app/outbox/**`, `backend/app/workers/**`, `backend/migrations/**`, `backend/tests/jobs/**`; queue/IaC only after approval.
- **Dependencies:** M3 and M4; queue/outbox semantics, retention, retry/cancel policy, concurrency and spend caps.
- **Tasks:** Define state machine and transition constraints; create idempotent job admission; persist job plus transactional outbox; add dispatcher and provider-neutral worker contract; implement bounded retry/backoff and lease expiry/orphan reconciliation; support cancel and terminal errors; expose owner-scoped status.
- **Acceptance:** Duplicate idempotency key does not create a second job; queue outage does not lose committed work; retries are bounded; cancellation and worker death converge to documented states; no request handler blocks on inference; user cannot choose provider/worker.
- **Tests:** State transition, idempotency, outbox replay/duplicate delivery, queue saturation/outage, lease expiry, retry classification, cancel race, cross-user status, orphan reconciliation.
- **Risks:** Queue technology/operator burden and exactly-once misconceptions; avoid Redis unless evidence justifies it.
- **Complexity:** L.
- **Rollback:** Stop dispatching new work; retain database job history and reconcile queued jobs explicitly. Schema changes remain backward-compatible.

### M6 — Image Processing Integration

- **Objective:** Integrate one legally approved, pinned image model through a headless/versioned engine interface and isolated worker; image-to-image only.
- **Files:** Proposed `ai-engine/**` or another approved isolated engine package, adapter code under `backend/app/engine/**` and/or `gpu-workers/**`, model/license inventory and tests. Preserve `modules/` and `run.py` unless a separately reviewed extraction task specifically changes them.
- **Dependencies:** M5; legal review of code, model weights and distribution; consent/abuse policy; approved engine/runtime compatibility; resource and cost budget; CPU/GPU provider decision.
- **Tasks:** Inspect current engine import graph and Qt/global-state coupling; establish versioned image engine contract; choose model only after rights clearance; pin model digest/runtime; process bounded input in isolated worker; upload private output; implement timeouts, cancellation and deterministic cleanup; record aggregate metrics only.
- **Acceptance:** Demonstrably transformed output from the approved model on supported inputs; no GUI/physical camera needed; model/license provenance recorded; input ownership/consent rechecked; worker has strict time/memory/input limits; no private media or embeddings in logs; no silent remote fallback.
- **Tests:** Real integration test using synthetic/authorized licensed fixtures; input/output quality baseline; model unavailable/corrupt, timeout, memory failure, cancellation, duplicate task, worker crash, cleanup, CPU/GPU capability and license manifest checks.
- **Risks:** Current model rights may prohibit commercial or hosted use; current code may not run headlessly; cloud egress/GPU costs; model fetch/redistribution obligations.
- **Complexity:** XL.
- **Rollback:** Disable image capability and stop dispatch; preserve job records and private media per approved policy; remove worker deployment only after sessions/jobs drain. Do not delete model/user data without authorization.

### M7 — Results & User Workflow

- **Objective:** Complete the authenticated user journey: submit, follow progress, cancel, view/download result, and manage job history.
- **Files:** Proposed `frontend/src/features/image/**`, `frontend/src/features/history/**`, `frontend/src/lib/api/**`, backend result/history routes, `frontend/tests/e2e/**`.
- **Dependencies:** M1–M6; finalized API contract and retention/download policy.
- **Tasks:** Connect the UI to real capabilities/API; show upload/job progress, retries and failure codes; prevent duplicate submissions; display safe preview and expiring authorized download; history pagination and deletion/expiry states; implement empty/offline/worker-unavailable states.
- **Acceptance:** A real authorized image is processed end-to-end; status reflects backend state; result only visible to its owner; expired/deleted result cannot be fetched; no fake success or provider internals exposed; mobile layout and keyboard accessibility pass.
- **Tests:** E2E auth/upload/consent/job success/failure/cancel/history/expired link, cross-user denial, upload interruption, API outage, mobile viewport, accessibility.
- **Risks:** Race between status events and refetch, result URLs leaking, retries that create duplicate jobs, premature deletion claims.
- **Complexity:** L.
- **Rollback:** Feature-flag/disable workflow and keep API/job records consistent; do not invalidate existing media except under approved retention policy.

### M8 — Security & QA

- **Objective:** Verify threat model, privacy, abuse resistance, compatibility, reliability, and performance for the scoped MVP.
- **Files:** Proposed `docs/implementation/phase-3/security/**` only if later requested; `frontend/tests/**`, `backend/tests/**`, `tests/contract/**`, `tests/e2e/**`, CI workflow changes after review.
- **Dependencies:** M1–M7; security/privacy review, approved test data, reference hardware and measurable budgets.
- **Tasks:** Threat-model upload/auth/job/result surfaces; dependency/image scans; IDOR/CSRF/CORS/rate-limit/quota checks; consent revocation and deletion verification; log/telemetry audit; load/worker failure tests; accessibility/mobile/browser matrix; operational runbooks.
- **Acceptance:** No unresolved critical/high findings; all user-owned operations have authorization tests; retention/deletion promises proven end-to-end; failure modes show truthful states; performance and cost limits measured and meet owner-approved targets; all existing desktop tests remain separately accounted for.
- **Tests:** Security suite, dependency audit, contract/E2E, mobile/browser matrix, load/fault injection, backup/restore and deletion evidence.
- **Risks:** Tests may reveal blockers requiring rework; privacy and model/license acceptance cannot be waived for schedule.
- **Complexity:** L.
- **Rollback:** Stop release; disable affected capability; fix forward or revert isolated release; preserve evidence and data for incident handling.

### M9 — Deployment

- **Objective:** Prepare a staging deployment and production-readiness assessment only after all gates; no production release or paid resource without separate approval.
- **Files:** Proposed `frontend/vercel.json` only if required; `backend/Dockerfile`, deployment/IaC under approved directory, CI workflow updates, runbooks and environment-variable name inventory. Exact paths/provider depend on approved architecture.
- **Dependencies:** M0–M8; Vercel root/framework correction; separate backend/worker hosting decision; regions, secrets, spend ceiling, data processing terms, incident/support owners.
- **Tasks:** Align Vercel project root with frontend app (or approved monorepo config); keep FastAPI, persistent queue workers, and WebRTC/GPU sessions on a runtime suited to long-lived tasks; configure staging-only secrets by platform; deploy non-production; test migrations/health/alerts/backups/restore/rollback; assess readiness.
- **Acceptance:** PR preview builds correct frontend root; backend and worker deploy independently; no inference or persistent streaming depends on an incompatible short-lived function; staging access and private media controls verified; alarms, spend cap, rollback and incident owner documented; production remains off until explicit approval.
- **Tests:** CI on clean checkout; frontend build/e2e; backend unit/integration; migration; storage privacy; end-to-end staging; alert/drain/rollback/restore drills.
- **Risks:** Current Vercel project is Python-rooted and failed; changing project settings may affect deployment; hosting GPUs, object storage and network transfer can incur charges.
- **Complexity:** L/XL.
- **Rollback:** Roll back staging deployment and migrations using documented method; stop worker spend; preserve data and audit state. Production deploy is a separate approval gate.

## Approval checkpoints

1. **Before M0 execution:** approve owners and audit scope; do not modify app or provision resources.
2. **Before M1:** approve MVP, frontend target, design/accessibility baseline, and allowed non-production API fixtures.
3. **Before M3/M4/M5:** explicitly approve identity, region, consent, retention, storage, queue, limits, and cost controls.
4. **Before M6:** obtain documented model/code rights clearance and approve inference runtime/provider/spend.
5. **Before M9:** authorize staging resources; production remains a separate written decision.

See [pending decisions](TECHNICAL_DECISIONS_PENDING.md), [MVP scope](MVP_SCOPE.md), and [workflow](DEVELOPMENT_WORKFLOW.md).
