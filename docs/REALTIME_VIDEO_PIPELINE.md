# Real-time video pipeline

## Current implementation (verified by source inspection)

Live preview is implemented inside the Python/PySide6 desktop app, not in a web service. `modules/ui.py` creates a `_CaptureWorker` and `_ProcessingWorker`; capture and processed queues are both bounded to two frames and discard an older frame when full. The processing loop reads the current source image, periodically detects faces, invokes frame processors, optionally sends output to the virtual camera, and queues a preview frame for the UI. This latest-frame policy limits backlog but does not by itself measure frame age or guarantee a latency bound.

`modules/video_capture.py` opens OpenCV camera devices and measures a startup FPS estimate. `modules/face_analyser.py` provides detection-only fast paths and optional landmark completion. `modules/processors/frame/face_swapper.py` includes InsightFace analysis/swap sessions, model selection, paste-back and optional image adjustments. The live loop uses a detection interval of approximately 80 ms (`round(camera_fps * 0.08)` frames); this is periodic detection, not face tracking. The `ROADMAP.md` likewise lists face tracking as future work.

`modules/virtual_camera.py` targets a fixed 1280x720, 30 FPS `pyvirtualcam` output. The current source uses OBS-provided virtual camera setup per its docstring/README. This is not evidence of a native Windows camera driver implementation or macOS CoreMediaIO extension maintained by this repository.

## Proposed target pipeline

1. **Opt-in camera session**: explicit start, visible active state, selected camera and resolution; acquire only after user action.
2. **Capture**: one owner per camera stream; timestamp frames at acquisition; release the device on stop, error, window close and application exit.
3. **Bounded latest-frame handoff**: small bounded queues; discard stale frames under overload and count every drop. Measure capture-to-consume age rather than queue depth alone.
4. **Face detection/tracking**: detect, associate detections with stable tracks, validate track confidence, and periodically refresh detection. Reset on camera/scene/source changes; never reuse stale coordinates indefinitely.
5. **Landmarks/alignment**: compute only required landmarks, align to the selected model, and reject invalid/low-confidence geometry safely.
6. **Source identity**: compute/load the authorized source embedding once per source revision and reuse it; source changes invalidate the cache explicitly.
7. **Inference**: long-lived, reusable ONNX sessions on the selected execution provider; avoid session initialization in the frame loop; make CPU/GPU transfers explicit and measured.
8. **Compositing/render**: mask, blend and color match; use temporal tracking/smoothing only when measured visual stability improves without unacceptable lag. Do not use simple frame averaging as a tracking substitute.
9. **Frame pacing/output**: pace to the device's supported output rate, define behavior when inference is slower (latest frame, bounded repeats or explicit degraded state), and deliver through a replaceable virtual-camera adapter.
10. **Preview/diagnostics**: display preview and per-stage metrics without blocking inference; redact media and sensitive identifiers from logs.

## Boundaries and ownership

Keep the frame engine independent from PySide6 and camera APIs behind interfaces such as `CameraSource`, `FaceSwapEngine`, and `VideoOutput`. This repository can own the local native agent and reusable Python engine. A future web dashboard may configure a paired local agent, but should not capture a webcam in the browser and imply that this output becomes WhatsApp's camera. A hosted backend/GPU worker is a separate product path and is not needed for local real-time calls.

## Reliability and privacy requirements

- Camera capture begins only after an explicit user action; visibly distinguish camera preview from transformed output.
- Stop camera tracks/device handles and virtual-camera output deterministically, including partial-start failures.
- On detector/inference/output exception, report a visible recoverable error, stop or enter a defined safe degraded mode, and avoid silently displaying stale transformed output.
- Bound queues, retry counts, model initialization, memory use, and shutdown waits. Do not turn transient frame failure into an unbounded retry loop.
- Require the user to have authorization for source and target likenesses; explain local processing and any network access. Do not add recording, cloud upload, or retention without an explicit design and consent.

## Status and validation

Architecture proposal only. Current code provides a functional local capture → detect → swap → preview / virtual-camera path, but this audit has not run a camera, GPU inference, WhatsApp call, or sustained-session test. See [performance targets](PERFORMANCE_AND_LATENCY_TARGETS.md) and [implementation plan](REALTIME_ENGINE_IMPLEMENTATION_PLAN.md).
