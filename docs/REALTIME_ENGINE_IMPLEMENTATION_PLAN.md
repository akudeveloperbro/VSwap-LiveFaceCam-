# Real-time engine implementation plan

## Goal

Evolve the existing local Python desktop pipeline into a measured, reliable, low-latency camera-to-virtual-camera engine, then verify compatibility with named WhatsApp clients. This is a proposal only. No application source code, camera behavior, model, backend, or integration was changed as part of this audit.

## Existing assets to preserve

- **Camera capture:** `modules/video_capture.py` already wraps OpenCV device opening, actual resolution/FPS reporting and release.
- **Bounded latest-frame flow:** `_CaptureWorker` and `_ProcessingWorker` in `modules/ui.py` use size-two queues and replace old frames on overflow.
- **Detection/inference:** `modules/face_analyser.py` contains detection-only paths and optional landmarks; `modules/processors/frame/face_swapper.py` owns swap models, paste-back, optional enhancement and adjustment logic.
- **Output adapter:** `modules/virtual_camera.py` provides a small `pyvirtualcam` abstraction with letterboxing and close behavior.
- **Existing validation seed:** `tests/test_ui_live_preview.py` covers selected UI-worker shutdown/source-loading control flow; `benchmark_pipeline.py` measures a subset of pipeline stages.

Preserve the local desktop-first boundary. Extract interfaces incrementally; do not rewrite the current face engine or move inference into a web/browser runtime merely to add a dashboard.

## Gaps and risks found

- Live detection is periodic (~80 ms), but there is no identity/landmark tracker; the roadmap explicitly lists tracking as future work.
- Queue bounds/drop-oldest are present, but capture timestamps, frame-age reporting, output FPS/drop metrics, overload status and end-to-end local latency measurement are not evident in the live loop.
- Current virtual output is fixed 1280x720@30, and depends on platform/OBS setup; no native driver or WhatsApp client test is present.
- Camera startup/FPS probing and lifecycle deserve adversarial tests; `ROADMAP.md` lists a blocking camera probe issue.
- Worker failure, camera disconnect, source changes, stop/restart and sustained memory/resource behavior need explicit coverage beyond the current tests.
- The benchmark uses InSwapper and 1080p camera assumptions and is not a current, reproducible 720p30 WhatsApp baseline.
- No WhatsApp integration, web dashboard, backend/auth/storage service, browser transformed track, or mobile implementation was found. Do not introduce these without an approved product decision.
- Model/data licenses differ; preserve current licensing notices and review model license before changing model distribution or commercial claims.

## Sequenced implementation

### 0. Product and reference-device decision

Choose launch OS, reference machine/GPU, camera, model, WhatsApp Desktop/Web build, target privacy model, and whether product remains local-first. Confirm the proposed 720p30 and p95 100 ms local frame-age gate in [performance targets](PERFORMANCE_AND_LATENCY_TARGETS.md). These choices determine what can actually be measured and supported.

**Exit:** signed-off compatibility matrix and benchmark protocol; no new paid infrastructure.

### 1. Establish baseline and strengthen observability

Add a reproducible benchmark path matching the application-selected model/provider. Timestamp capture and output using a monotonic clock; measure detection, optional landmarks, inference, compositing, conversion, output send, queue age, drops, input/output FPS, memory and errors. Use aggregate local telemetry only. Record baseline runs before algorithm changes.

**Exit:** three comparable runs on the reference system; p50/p95 stage and frame-age results; known bottleneck identified.

### 2. Harden capture and worker lifecycle

Move potentially blocking camera open/probe off the UI thread, define partial-start cleanup, camera disconnect and device-busy recovery, worker exception propagation, cancellation and deterministic stop. Keep bounded queues and latest-frame behavior; add observable drop/overload status. Add tests with slow/failed camera and processor stubs.

**Exit:** no UI freeze on stalled camera; all workers/resources closed after normal, failed and interrupted starts; queues stay bounded under overload.

### 3. Extract stable engine interfaces

Separate camera source, engine configuration, `FaceSwapEngine` (source embedding, detect/track, process), and `VideoOutput` from PySide UI. Move in small reversible seams and retain existing implementations as adapters. Ensure reusable sessions and source-revision-based cache invalidation; explicit provider/model capability errors.

**Exit:** engine control-flow tests run without Qt or physical camera; existing desktop behavior remains available.

### 4. Add tracking and temporal stability

Evaluate a bounded tracking/association method between detector refreshes; use confidence and lost-track timeouts, smooth landmarks with lag bounds, and force redetection on discontinuities. Compare flicker, landmark jitter, head-turn recovery, occlusion and p95 latency to the baseline. Do not use simple frame averaging as a substitute for tracking.

**Exit:** measurable stability improvement with no violation of frame-age target and safe fallback to fresh detection.

### 5. Validate output adapters and client compatibility

Keep the current `pyvirtualcam`/OBS route first. Verify OS device enumeration, format/rate handling, send failure, multiple consumers and cleanup. Run the named WhatsApp Desktop and Web matrix through supported camera selection and an authorized test call. Native Windows/macOS drivers require separate technical, distribution and security review.

**Exit:** each advertised OS/client matrix row passes sustained call and recovery tests; otherwise mark unsupported.

### 6. Soak, release gate and deferred directions

Run 30-minute sessions, overload/disconnect/restart tests, 720p30 frame pacing and visual quality review. Validate model/license notices and consent text. Decide whether 60 FPS, mobile, web dashboard or remote processing merits separate approval. Document actual support and measured constraints.

**Exit:** reproducible evidence against [performance targets](PERFORMANCE_AND_LATENCY_TARGETS.md), privacy review and support matrix; no marketing claim beyond evidence.

## Decisions requiring approval

- Local desktop product vs. hosted inference architecture (this plan assumes local desktop processing).
- Supported OS/client/reference-device matrix and acceptance thresholds.
- Whether to fund/maintain native camera drivers or an independently designed web control plane.
- Any mobile app, cloud storage, account, subscription, remote GPU, or new paid infrastructure.

## Explicit non-goals for this phase

WhatsApp code injection, undocumented/private APIs, encrypted communication interception, mobile compatibility claims, audio capture/routing, remote GPU/backend migration, model replacement, and production implementation before approval.

## Related assessments

- [WhatsApp compatibility](WHATSAPP_COMPATIBILITY_ASSESSMENT.md)
- [Real-time pipeline](REALTIME_VIDEO_PIPELINE.md)
- [Virtual-camera architecture](VIRTUAL_CAMERA_ARCHITECTURE.md)
- [Performance targets](PERFORMANCE_AND_LATENCY_TARGETS.md)
- [Mobile compatibility](MOBILE_COMPATIBILITY_ASSESSMENT.md)
