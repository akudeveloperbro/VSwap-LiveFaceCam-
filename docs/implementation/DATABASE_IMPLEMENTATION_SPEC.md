# PostgreSQL Database Implementation Specification

**Status:** Proposed logical schema, not a migration or current database. PostgreSQL is recommended for durable relational state; exact auth provider, tenancy model, retention and ORM remain decisions. Candidate: SQLAlchemy 2.x + Alembic for Python migrations.

## Conventions and relationships

Use UUID primary keys generated server-side; `timestamptz` UTC timestamps; `jsonb` only for validated versioned metadata, not as a substitute for relational columns. Store money as integer minor units plus ISO currency, never floating point. Use foreign keys, unique constraints and explicit state enums/check constraints. Every user-owned row carries `user_id` or is reachable through a user-owned parent, and every query enforces owner/tenant scope. Recommend soft lifecycle states only where audit/compliance requires them; physical media deletion is separate and verified.

Relationship overview: users 1—N settings, source_faces, media_assets, jobs, sessions, subscriptions, usage; processing_jobs and processing_sessions may reference media_assets and a source_face; sessions/jobs may have worker leases; providers 1—N workers; usage references user and optional job/session; audit logs reference actor/target IDs without raw media; plans 1—N subscriptions. Create tables in dependency order: users, gpu_providers, subscription_plans, media_assets, consent_records, user_settings, source_faces, gpu_workers, processing_sessions, processing_jobs, subscriptions, usage_records, audit_logs (adjust circular references using nullable columns and a later constraint migration if needed).

## Tables (logical fields; all structures are proposals)

### `users`

- `id UUID PK`; `email CITEXT UNIQUE NOT NULL`; `display_name TEXT NULL`; `status TEXT NOT NULL CHECK`; `created_at`, `updated_at`, `deleted_at TIMESTAMPTZ NULL`.
- If using an external/auth library, credentials/session secrets may belong in its owned schema; do not duplicate password hashes without a decision.
- Index: unique normalized email; status. Security: account enumeration-resistant auth responses; deletion and retention policy.

### `user_settings`

- `user_id UUID PK/FK users(id) ON DELETE CASCADE`; `locale TEXT`; `theme TEXT`; `preferred_mode TEXT`; `remote_processing_allowed BOOLEAN NOT NULL DEFAULT false`; `settings JSONB NOT NULL DEFAULT '{}'`; `updated_at TIMESTAMPTZ`.
- Index: none beyond PK. Keep only allowlisted preferences; never store credentials or consent proof only as a mutable setting.

### `consent_records` (recommended additional table)

- `id UUID PK`; `user_id UUID FK users`; `purpose TEXT`; `subject_asset_id UUID NULL FK media_assets`; `policy_version TEXT`; `consent_status TEXT`; `captured_at`, `revoked_at TIMESTAMPTZ NULL`; `evidence JSONB` minimal/approved.
- Indexes: `(user_id, captured_at DESC)`, `(consent_status, revoked_at)`. A mutable profile flag is not adequate evidence for a specific remote transformation consent.

### `source_faces`

- `id UUID PK`; `user_id UUID FK users`; `asset_id UUID FK media_assets`; `label TEXT`; `consent_status TEXT`; `consent_record_id UUID NULL FK consent_records(id)`; `created_at`, `updated_at`, `deleted_at`.
- Indexes: `(user_id, created_at DESC)`, `(user_id, id)`; unique asset/source relation if one face per asset policy applies.
- Do not persist face embeddings by default. If later required, separate encrypted biometric data, key rotation, retention/consent/deletion and access review. Avoid writing identity/biometric labels to logs.

### `processing_sessions`

- `id UUID PK`; `user_id UUID FK users`; `source_face_id UUID NULL FK source_faces`; `mode TEXT`; `status TEXT`; `engine_version TEXT NULL`; `model_version TEXT NULL`; `provider_id UUID NULL FK gpu_providers`; `worker_id UUID NULL FK gpu_workers`; `region TEXT NULL`; `config JSONB` (validated, non-secret); `started_at`, `ended_at`, `expires_at`, `created_at TIMESTAMPTZ`.
- Indexes: `(user_id, created_at DESC)`, `(status, expires_at)`, `(worker_id, status)`. Store no raw frames or signaling credentials.

### `processing_jobs`

- `id UUID PK`; `user_id UUID FK users`; `source_face_id UUID NULL FK source_faces`; `input_asset_id UUID FK media_assets`; `output_asset_id UUID NULL FK media_assets`; `kind TEXT CHECK image/video`; `status TEXT`; `progress SMALLINT NULL CHECK 0..100`; `idempotency_key TEXT`; `engine_version TEXT NULL`; `model_version TEXT NULL`; `options JSONB`; `error_code TEXT NULL`; `created_at`, `started_at`, `finished_at`, `expires_at`.
- Indexes: unique `(user_id, idempotency_key)` where key non-null; `(user_id, created_at DESC)`, `(status, created_at)`, `(expires_at)`. Never expose cross-user existence through status lookups.

### `gpu_providers`

- `id UUID PK`; `provider_key TEXT UNIQUE`; `display_name TEXT`; `status TEXT`; `regions JSONB`; `capabilities JSONB`; `created_at`, `updated_at`.
- Indexes: `(status, provider_key)`. Admin/service role only. Credentials live in secret manager, not columns.

### `gpu_workers`

- `id UUID PK`; `provider_id UUID FK gpu_providers`; `provider_resource_ref TEXT`; `status TEXT`; `region TEXT`; `gpu_type TEXT NULL`; `engine_version TEXT NULL`; `capacity JSONB`; `lease_expires_at TIMESTAMPTZ NULL`; `last_heartbeat_at TIMESTAMPTZ NULL`; `created_at`, `updated_at`, `terminated_at`.
- Unique `(provider_id, provider_resource_ref)`; indexes `(status, region)`, `(lease_expires_at)`. Treat provider references as sensitive infrastructure metadata; restrict to orchestration/admin roles.

### `usage_records`

- `id UUID PK`; `user_id UUID FK users`; `job_id UUID NULL FK processing_jobs`; `session_id UUID NULL FK processing_sessions`; `metric TEXT`; `quantity BIGINT CHECK >=0`; `unit TEXT`; `cost_minor BIGINT NULL CHECK >=0`; `currency CHAR(3) NULL`; `source TEXT`; `occurred_at TIMESTAMPTZ`; `metadata JSONB` allowlisted.
- Indexes: `(user_id, occurred_at DESC)`, `(job_id)`, `(session_id)`, optional provider reconciliation key unique. Append-only; do not make client-reported usage authoritative.

### `subscription_plans`

- `id UUID PK`; `plan_key TEXT UNIQUE`; `display_name TEXT`; `status TEXT`; `limits JSONB`; `price_minor BIGINT NULL`; `currency CHAR(3) NULL`; `billing_interval TEXT NULL`; `created_at`, `updated_at`.
- Index `(status, plan_key)`. Price/entitlement source and payment processor remain undecided.

### `subscriptions`

- `id UUID PK`; `user_id UUID FK users`; `plan_id UUID FK subscription_plans`; `provider TEXT`; `provider_subscription_ref TEXT NULL`; `status TEXT`; `period_start`, `period_end`, `cancel_at`, `created_at`, `updated_at` TIMESTAMPTZ.
- Indexes `(user_id, status)`, unique `(provider, provider_subscription_ref)` where non-null. Do not trust browser plan/price fields; webhook reconciliation and idempotency required.

### `media_assets`

- `id UUID PK`; `user_id UUID FK users`; `kind TEXT`; `object_key TEXT UNIQUE`; `mime_type TEXT`; `byte_size BIGINT CHECK >=0`; `duration_ms BIGINT NULL`; `width INTEGER NULL`; `height INTEGER NULL`; `sha256 BYTEA`; `status TEXT`; `retention_until TIMESTAMPTZ`; `created_at`, `deleted_at`.
- Indexes `(user_id, created_at DESC)`, `(status, retention_until)`. Do not store public URLs or signed URLs as durable data. Object-store metadata must be private; cleanup job reconciles DB and object state.

### `audit_logs`

- `id UUID PK`; `actor_user_id UUID NULL FK users`; `action TEXT`; `target_type TEXT`; `target_id UUID NULL`; `request_id TEXT`; `result TEXT`; `occurred_at TIMESTAMPTZ`; `metadata JSONB` allowlisted and redacted.
- Indexes `(actor_user_id, occurred_at DESC)`, `(target_type, target_id)`, `(occurred_at)`. Append-only restricted role; no face media, tokens, full IP unless approved retention/security policy requires it.

### `consent_records` (recommended additional table)

- `id UUID PK`; `user_id UUID FK users`; `purpose TEXT`; `subject/asset reference UUID NULL`; `policy_version TEXT`; `consent_status TEXT`; `captured_at`, `revoked_at` TIMESTAMPTZ NULL; `evidence JSONB` minimal/approved.
- Indexes `(user_id, captured_at DESC)`, `(consent_status, revoked_at)`. A mutable profile flag is not adequate evidence for a specific remote transformation consent.

## Migrations, isolation, retention

Use reviewed, reversible Alembic migrations with expand/contract for production changes, backups and rollback plan. Enforce authorization in service queries; consider PostgreSQL RLS only as defense-in-depth after proving connection-pool tenant context cannot leak. Data retention must define original/input/output deletion, temporary artifacts, audit/usage retention and backups. Delete storage objects asynchronously with retries and visible completion status; never report deletion before confirmation. No schema should be applied until owner, auth, tenancy, consent and retention decisions are approved.
