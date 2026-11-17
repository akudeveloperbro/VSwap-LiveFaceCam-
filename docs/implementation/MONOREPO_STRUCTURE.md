# Proposed Monorepo Structure

**Status:** Proposal only. Do not move, delete, or rename current files in this phase. Existing repository remains intact.

## Target layout

```text
.
├── frontend/                  # Next.js App Router, design system, browser tests
├── backend/                   # FastAPI control plane, migrations, API/integration tests
├── ai-engine/                 # Versioned reusable Python engine package (extracted incrementally)
├── gpu-workers/               # Worker entrypoints, container recipes, provider-neutral runtime
├── native-camera/             # OS-specific virtual camera/agent experiments and packaging
├── shared/                    # Versioned API schemas, generated types, error/status enums
├── infrastructure/            # IaC, deployment/network/storage/monitoring; only after approval
├── docs/                      # Architecture, implementation specs, operations, decisions
├── tests/                     # Cross-boundary contract, E2E, performance and security harnesses (added after approval)
├── modules/                   # Existing LiveFaceCam code preserved during migration
├── requirements.txt           # Existing desktop dependency manifest; do not overwrite
└── README.md                  # Existing desktop product documentation; reconcile later
```

The current repository's existing root tests remain in place during this phase. After approval, add cross-boundary tests under named subdirectories (for example, `tests/contract/`, `tests/e2e/`, `tests/performance/`) and move legacy tests only in a separately reviewed migration that preserves history and runnable paths.

## Directory responsibilities

- `frontend/`: route groups, layouts, components by feature, typed API client, WebSocket/WebRTC client adapters, accessibility and browser tests. No inference models, provider SDKs or secret media URLs.
- `backend/`: API routers, auth adapter, domain services, repository/data access, Pydantic schemas, SQLAlchemy models, Alembic migrations, object-storage adapter, queue adapter, orchestration and contract tests. Keep provider SDK integration behind `backend/providers/`.
- `ai-engine/`: inference/session protocol, detection/tracking/alignment/swap/blend/enhance/render stages, runtime capability and unit/quality tests. Engine must not import FastAPI, provider SDKs, billing or UI.
- `gpu-workers/`: worker process lifecycle, job/session adapters, health/readiness, resource limits, image build, structured aggregate telemetry and fault tests. No durable user database ownership.
- `native-camera/`: Windows/macOS/mobile feasibility and eventual native adapters, separated by OS. Experimental until tested and approved; no WhatsApp hooks.
- `shared/`: OpenAPI source/schema, event payload definitions and generated frontend type bindings; avoid shared runtime code that couples Python and TS unnecessarily.
- `infrastructure/`: network policy, secret references, private object storage, database, worker compute, metrics/alerts, budgets. Never commit secret values or deploy without approval.
- `docs/`: source-of-truth architecture, decision register, threat model, lifecycle and runbooks.
- `tests/`: cross-service contracts, E2E, load, security, WebRTC/network and compatibility evidence.

## Safe migration strategy

1. Keep the existing checkout and all desktop source/assets/tests untouched while decisions are unresolved.
2. Add new apps/services as separate roots only after approval; pin their dependencies independently and avoid changing root `requirements.txt` for web/backend packages.
3. Define engine API and tests before moving modules. Introduce adapters around existing code, run legacy tests unchanged, then migrate files in small reviewable steps with path compatibility and license notices preserved.
4. Split the project into separate repositories only if ownership, release cadence, CI, model-weight access or security boundaries require it; preserve commit history and document the exact migration.
5. Do not copy model weights, user media, secrets, generated outputs or environment files into the monorepo.

## CI and ownership

Path-filtered CI should validate frontend, backend, engine and worker independently, with a cross-boundary contract suite. Changes to `shared/` trigger all consumers. Keep model-weight tests opt-in and license-approved. Native builds run only on supported host runners. Current repository has Python CI workflows; no proposed TypeScript/Python services or infrastructure exist yet.
