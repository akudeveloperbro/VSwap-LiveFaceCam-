# M0 — Repository & Development Environment Checklist

**Status:** Prepared for a future, explicitly approved M0. M0 execution has not started. The Phase 3 planning audit reviewed the repository and deployment evidence, but did not change application code, modify environment/configuration, run the application test suite, provision resources, or alter Vercel. Checked boxes record facts confirmed during planning only; they do not mark M0 complete.

## A. Repository integrity and ownership

- [ ] Confirm approved feature branch and base commit; verify a clean working tree and preserve all pre-existing edits.
- [x] Planning audit observed current branch `docs/implementation-blueprint` at `daa2c100a2d6d7f4e59c5049ea71722ff2080282`; `main`/`origin/main` at `a901c5d36de9107a8726330b89897e9e956626d3`; no unrelated dirty edits were observed during the audit.
- [x] PR #1 is merged; PR #2 is open, not merged, at the current blueprint commit. Record current status again before M0 begins.
- [ ] Review latest PR checks/review feedback, merge conflicts, and concurrent commits before any branch work.
- [ ] Preserve the Python desktop product, its history, entrypoints, dependencies, assets, locale files, installers, and tests.

## B. Structure, manifests, setup, and runtime

- [x] Inventory shows Python desktop source under `modules/`, root `run.py`, PySide6 UI, existing local image/video processing, and `pyvirtualcam` integration.
- [x] Root `requirements.txt` is desktop/inference oriented; `pyproject.toml` contains Ruff settings only; no Node package manifest, Node lockfile, frontend, FastAPI service, Vercel config file, Docker/Compose, or database/queue code was found.
- [x] Existing Windows setup includes `Setup.bat` model downloads and venv install; README documents manual multi-OS setup. These paths must not be overwritten by web setup.
- [ ] On a clean supported environment, follow documented desktop setup without user media and record Python/OS/CPU/GPU/runtime/provider, install result and warnings.
- [ ] Choose isolated `frontend/` and `backend/` dependency manifests, pinned package managers/lockfiles and supported runtime versions; verify no root dependency collisions.
- [ ] Document exact local dev commands, required non-secret environment-variable names, health checks and recovery. Never commit secrets or sample personal media.

## C. Existing tests and CI

- [x] Six Python test modules exist under `tests/`; two CI workflows run Python compileall and Ruff. Neither workflow currently runs the tests.
- [ ] Execute existing test suite using repository-supported dependencies, record exact command, versions, pass/fail/skip and platform limitations. Do not treat compile/lint as tests.
- [ ] Decide whether existing test command is unittest/pytest and whether a test dependency is already present before adding anything.
- [ ] Establish proposed independent frontend/backend checks and path-filtered CI without changing existing CI until separately approved.
- [ ] Baseline legacy desktop smoke/CLI tests and document which require OS/GPU/model files; avoid downloading or committing licensed weights absent approval.

## D. Blueprint, scope and unfinished work

- [x] Read Phase 2 master, frontend/API/backend/database/engine/GPU/monorepo/testing specs, decision register, and the existing web platform roadmap.
- [x] Record conflict risk: broad Phase 2 documents describe real-time/WebRTC/native/WhatsApp futures; proposed MVP is narrower, image-only async processing. Keep those features out unless separately approved.
- [x] Record `ROADMAP.md` open desktop bugs/features as a separate maintenance track; none should be silently merged into web work.
- [x] Record existing model-license warning: some current model weights are research/non-commercial; intended hosted use is not cleared by the README or by architectural recommendation.
- [ ] Approve written MVP scope, image limits, supported users/regions, and explicit exclusion list.
- [ ] Clear code/model/weight license and redistribution/hosting rights before inference implementation.
- [ ] Approve auth/session, tenancy, consent/abuse, retention/deletion, processing region, cost ceilings, provider and hosting decisions.

## E. CI/CD and Vercel

- [x] Existing repository has `python-compile.yml` and `ruff.yml`; no frontend/backend tests or deployment workflow are present.
- [x] Vercel project inspection reports root `.`, Python framework preset, no build command, and install `pip install -r requirements.txt`.
- [x] PR #2 Vercel build log reports `[PYTHON_ENTRYPOINT_NOT_FOUND]`; the project is configured as Python even though current root is a desktop app with no Vercel Python web entrypoint. Vercel Preview Comments passes.
- [ ] Decide the approved frontend root and Vercel framework/build/install settings after the app exists; do not “fix” settings by pointing at an empty path.
- [ ] Decide independent API, worker, storage and database hosting; confirm long-lived worker/WebRTC limitations and no GPU inference in short-lived frontend functions.
- [ ] Test preview deployment on a non-production change after Vercel settings change is explicitly approved. Confirm production alias remains untouched.

## F. Exit criteria and handoff

- [ ] Clean checkout can reproduce the approved existing desktop baseline and new-service setup independently.
- [ ] Existing CI/test results are recorded; known failures and gaps have owners or explicit deferrals.
- [ ] Package managers/lockfiles, supported runtimes, branch/CI conventions and service boundaries are documented.
- [ ] P0 decisions in [pending decisions](TECHNICAL_DECISIONS_PENDING.md) are resolved or explicitly block the dependent milestone.
- [ ] Architecture/docs are internally consistent; links and path references validate; no secrets or personal media are present.
- [ ] Owner explicitly accepts M0 evidence and separately authorizes M1. M0 completion does not authorize M1, paid infrastructure or production deployment.

## Audit notes (as observed for this plan)

- PR #2 is open and Vercel deployment check failed; deployment failure is traced to the project’s Python preset/root setup rather than a failing Markdown document. No repository-side Vercel config exists to edit.
- The current Phase 2 [master blueprint](../MASTER_IMPLEMENTATION_BLUEPRINT.md) proposes M1 frontend and later batch image processing; this Phase 3 plan keeps the user-requested M0–M9 sequence while limiting initial inference to one still image.
- The planning audit is documentation work only. No existing tests were executed and no application code or environment was modified during the audit.
