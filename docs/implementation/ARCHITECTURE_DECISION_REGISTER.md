# Architecture Decision Register

**Status:** Open decision log for the proposed VSwap web platform. No unresolved item below should be silently treated as approved. Owners and dates are not assigned because none were supplied. Updated 2026-11-18.

| ID | Decision needed | Current recommendation / default for evaluation | Impact and required evidence | Status |
|---|---|---|---|---|
| ADR-001 | MVP operation order: image, video, live | Start with image batch, then video, then live; validate product priority. | Scope, testability, cost and engine readiness. | Open |
| ADR-002 | Auth/session provider and identity model | Choose maintained auth; backend authoritative; avoid custom password handling unless required. | Sessions, user schema, CSRF, account deletion, frontend protection. | Open |
| ADR-003 | User tenancy model | Single-user ownership initially, retain tenant-ready data boundaries only if organization use is approved. | Authorization/query design and schema. | Open |
| ADR-004 | Backend framework | FastAPI candidate due to Python engine adjacency; API and worker processes remain separate. | Team ownership, deployment support, observability and load tests. | Proposed, not approved |
| ADR-005 | ORM/migration method | SQLAlchemy 2.x + Alembic candidate; direct SQL remains viable for small data access layer. | Schema review, migration rollback and team skill. | Open |
| ADR-006 | Durable job/event mechanism | PostgreSQL state + transactional outbox; Redis only for justified ephemeral/queue needs. | Delivery/retry/order guarantees, operations, load and failure semantics. | Open |
| ADR-007 | Private object storage/provider/region | Choose after data residency, signed access, deletion and cost review; no vendor assumed. | Media privacy, egress, durability and retention. | Open |
| ADR-008 | Model/weight distribution rights | Audit every code/model license and source before cloud deployment or product monetization. | May block commercial use, packaging, redistribution or model selection. | Blocker pending review |
| ADR-009 | Local CPU definition | Distinguish browser WASM/WebGPU path from installed desktop agent; evaluate independently. | Browser support, download size, memory, thermal and privacy. | Open |
| ADR-010 | Cloud GPU inclusion and first provider | No provider selected; compare RunPod, Vast.ai, Modal, Replicate with same workload and date-stamped terms. | Spend, regions, startup, persistence, streaming, security and lock-in. | Open |
| ADR-011 | Cloud budget and cost ceiling | Set per-user/session/job ceiling and global spend controls before provisioning. | Worker scale, idle pools, quotas and billing UX. | Open |
| ADR-012 | WebRTC topology | Decide media server/SFU/gateway/worker path and TURN after network prototype. | Latency, worker affinity, bandwidth, deployment and privacy. | Open |
| ADR-013 | Live target thresholds/reference hardware | Use proposed 720p30, p95 ≤100 ms frame-age gate only after device/protocol approval; 1080p60 future. | Product claims and performance release criteria. | Open |
| ADR-014 | Consent, abuse and synthetic disclosure | Define per-asset/session authorization, revocation, allowed use, reporting/enforcement and output disclosure. | Legal, safety, trust, UX and audit requirements. | Open |
| ADR-015 | Media retention/deletion/backups | Define originals, outputs, caches, worker scratch, audit and backup retention; deletion must be verified. | Schema, storage, operations, privacy and restore. | Open |
| ADR-016 | Source-face/embedding persistence | Do not persist embeddings by default; investigate whether reusable source faces can be represented by private source media only. | Sensitive biometric data and consent/legal obligations. | Proposed default |
| ADR-017 | Billing/subscription provider | Do not implement payment UI/entitlements as real until processor, plans, tax/refund and webhook behavior are selected. | Product/business and security. | Open |
| ADR-018 | WhatsApp support scope | Treat Desktop/Web/Android/iOS as distinct; no compatibility claim before transformed remote-call test. | Native components, OS restrictions, support matrix and release claims. | Open |
| ADR-019 | Native camera component strategy | Evaluate documented Windows/macOS APIs and mobile extension availability separately; no private hooks. | Signing, distribution, privileges, maintenance and security. | Open |
| ADR-020 | Repository/monorepo boundary | Preserve current repository; add isolated roots only after API/engine seams; split repos only with ownership rationale. | CI, licensing, model access, release cadence and blast radius. | Open |
| ADR-021 | First executable milestone | M1 responsive frontend shell using actual capability states; no fake successful work; then M2–M4 batch backend path. | Needs explicit authorization and contract fixture/real endpoint choice. | Proposed |

## Approval protocol

Record decision, alternatives, rationale, evidence, approver and date before changing status from Open/Proposed. Any decision involving paid infrastructure, personal/biometric data, model terms, public compatibility claims or production deployment requires explicit owner approval. See [master blueprint](MASTER_IMPLEMENTATION_BLUEPRINT.md) for the source-code gate.
