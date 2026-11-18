# Development Workflow and Guardrails

**Status:** Proposed mandatory protocol for future implementation. This plan and its approval are not permission to begin coding or spend money.

## Before a task

1. Confirm the user explicitly approved the milestone and the specific task. Approval of a roadmap, PR, or general direction does not approve paid services, production deployment, privacy policies, or infrastructure.
2. Read the current branch, working tree, repository instructions, owning code, callers, tests, and adjacent architecture docs. Do not rely on earlier snapshots if the current source can be inspected.
3. Start each milestone on a dedicated feature branch from the current approved base. Never commit directly to `main`/default branch; preserve unrelated changes and do not use force-push.
4. Write a one-task scope with acceptance checks and rollback. One milestone and one focused implementation task at a time; stop for a checkpoint before moving on.
5. Confirm there are no unresolved P0 decisions for the work. Stop rather than filling consent, data retention, region, model rights, provider, costs, auth, or billing assumptions silently.
6. Inspect package manifests and lockfiles. Add only necessary dependencies in the isolated service root; do not pollute or replace the desktop root dependencies.

## During implementation

1. Preserve `run.py`, desktop behavior, model download/licensing notices, root `requirements.txt`, `modules/`, assets, locales and existing tests unless the approved task specifically requires a reviewed change.
2. Follow boundaries: browser UI has no inference/provider credentials; API owns identity, authorization, consent, limits and job truth; workers handle bounded inference; engine is provider-agnostic; object storage is private. Never submit frames to REST per frame.
3. No mock authentication, fake jobs, canned success outputs, fabricated metrics, or UI states implying an unavailable service is working. Test fixtures stay test-only and are clearly identified.
4. Scope each data query and media operation to the authenticated owner/tenant. Validate media after decoding; enforce size, type, dimension, duration, quotas and cost on the server. Use parameterized queries and secure session practices.
5. Keep secrets out of Git, logs, URLs, fixtures, docs, screenshots and browser bundles. Read only environment-variable names when auditing; never print secret values.
6. Minimize personal/biometric data. Do not persist embeddings by default. Do not silently route local media to a cloud service; require explicit per-operation consent.
7. Do not provision, deploy, upgrade paid resources, enable public traffic, or change Vercel project settings without explicit approval of the exact action and limits.
8. Avoid unrelated refactors and unreviewed dependency/toolchain changes. Keep diffs small and explain any change to existing behavior.

## Testing and completion

1. Run focused tests for changed behavior, then applicable broader checks and existing desktop regressions. Tests must run before reporting code completion; record exact commands and outcomes. If unavailable, report blocker honestly—do not claim pass.
2. Test required negative paths: unauthenticated, unauthorized/cross-user, invalid media, consent revoked, capability unavailable, worker/queue/storage outage, timeout, cancellation, deletion and expired result as relevant.
3. For UI work, verify browser behavior at the current mobile viewport first, keyboard/accessibility, and applicable desktop width. No screenshots of personal media or secrets.
4. For data/schema changes, test migrations from clean and prior schema, rollback/forward recovery, ownership and backup/restore. Never use production media for test fixtures.
5. Review diff for unrelated files, secrets, license notices, false capability claims, broken relative links and generated artifacts. Report exact paths changed and checks/results.
6. Commit only task files on the feature branch. Push only the approved branch; open/update a PR unless asked otherwise. Never merge or deploy without explicit approval. Stop at the milestone checkpoint with open decisions, risks and next-task proposal.

## Approval gates

- **M0 → M1:** repository/development baseline and MVP decisions reviewed.
- **M1 → M2:** UI contract and capability states accepted.
- **M2 → M3:** API/error/auth boundary accepted.
- **M3 → M4:** identity, tenancy, region and retention accepted.
- **M4 → M5:** private upload/deletion and file limits accepted.
- **M5 → M6:** queue/retry/cost semantics accepted.
- **M6 → M7:** model/code rights, real inference safety and engine boundary accepted.
- **M8 → M9:** security/privacy/QA evidence accepted; staging spend separately approved.
- **Production:** separate explicit deployment approval, budget owner, operational owner and rollback plan required.

## Stop conditions

Stop and ask for a decision when a task needs unapproved data processing, unclear rights, external credentials, new paid services, destructive migration, public compatibility claim, auth bypass, production deployment, material schema/architecture change, or unresolved high-risk security finding. Preserve work and report the exact blocker; do not substitute a demo/mock implementation.

See [Phase 3 roadmap](PHASE_3_ROADMAP.md), [pending decisions](TECHNICAL_DECISIONS_PENDING.md), and [M0 checklist](MILESTONE_0_CHECKLIST.md).
