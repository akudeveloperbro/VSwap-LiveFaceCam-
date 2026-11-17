# Realtime engine implementation plan

## Goal and status

This document now aligns the engine workstream with the approved-in-principle web-first product direction. It is a proposal only. The existing LiveFaceCam repository is the AI engine foundation; it remains a Python/PySide6 desktop application today. No web backend, headless engine service, cloud worker, WebRTC service or provider integration was verified in the inspected source. Do not rewrite or modify the engine as part of this architecture-documentation revision.

## Verified engine foundation

- `modules/video_capture.py` wraps OpenCV device opening, reports actual resolution/FPS and releases the device.
- `modules/ui.py` contains `_CaptureWorker` and `_ProcessingWorker`; live capture and processing use bounded size-two queues that discard older frames under overflow.
- `modules/face_analyser.py` includes detection paths and optional landmarks; `modules/processors/frame/face_swapper.py` owns swap models, paste-back and optional enhancement/adjustment logic.
- `modules/virtual_camera.py` wraps `pyvirtualcam`, with letterboxing and close behavior; current README/docstrings describe the OBS virtual camera route.
- `benchmark_pipeline.py` measures a subset of a local webcam pipeline; its current parameters do not establish web/cloud performance.
- `ROADMAP.md` records uncompleted issues and proposed features. README/model terms include non-commercial/research restrictions for some models; review each model's terms before web or commercial deployment.

These are source/repository observations, not proof of cloud readiness, current deployment performance, rights to redistribute weights, or WhatsApp compatibility.

## Revised direction

Extract a reusable, versioned, headless Python engine boundary incrementally, retaining the desktop GUI as an existing adapter. Keep provider lifecycle, API authentication, media storage, and orchestration outside the engine. The future worker wraps a defined engine protocol and pins runtime/model versions. A browser-local CPU implementation is a separate feasibility investigation and must not be inferred from Python CPU support.

## Work sequence (proposal)

1. **Engine and license audit:** inventory import-time GUI coupling, model downloads, filesystem assumptions, provider selection, model licenses, media handling and deterministic cleanup. Define capability and version metadata.
2. **Contract design:** specify session open/process/close and batch asset semantics; input/output formats, timestamp propagation, limits, errors and health. No provider SDK in the engine API.
3. **Headless extraction spike:** introduce a narrow service/test harness while preserving desktop behavior; use stubs for camera and model dependencies. Keep changes reversible and separately approved.
4. **Worker packaging:** pin runtime and model artifacts by digest/version; run as least-privileged process with bounded memory, queue, CPU/GPU time, input dimensions and session lifetime.
5. **Performance and quality baseline:** benchmark representative authorized media and reference hardware; timestamp frames monotonically; report stage times, queue age/drops, FPS, p50/p95 frame age and memory. Do not use existing README FPS claims as a cloud baseline.
6. **Fault and isolation validation:** test cold model load, bad media, slow inference, worker cancellation/crash, concurrent tenant isolation, resource cleanup, and model/runtime mismatch.
7. **Adapter and provider work:** only after provider evaluation approval, integrate through the separate provider abstraction. Compare batch and persistent streaming use cases independently.

## Open technical blockers

- The current UI/worker implementation is not established as importable without Qt or camera devices.
- Model weight origin, model licenses, redistribution rights, and deployment inclusion need review.
- Existing local queue behavior does not prove end-to-end frame-age measurement, multi-tenant isolation, remote streaming or resource quotas.
- GPU memory requirements, model warmup, per-session concurrency, provider-specific device support and costs are unknown.
- Local desktop camera and virtual-camera lifecycle cannot be reused as browser or WhatsApp support without platform-specific work.

## Non-goals for this phase

No engine edits, model replacement, dependency installation, provider deployment, frontend/backend source implementation, WhatsApp compatibility claim, camera injection, or production performance claim. See [web roadmap](VSWAP_WEB_IMPLEMENTATION_ROADMAP.md), [worker architecture](CLOUD_GPU_WORKER_ARCHITECTURE.md), and [hybrid processing](HYBRID_PROCESSING_ARCHITECTURE.md).
