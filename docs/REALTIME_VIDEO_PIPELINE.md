# Realtime video pipeline assessment

## Current implementation (verified by source inspection)

The current live preview is inside the Python/PySide6 desktop application, not a web service. `modules/ui.py` creates `_CaptureWorker` and `_ProcessingWorker`; capture and processed queues are bounded to two frames and discard an older frame when full. Processing periodically detects faces, invokes frame processors, optionally sends output to the virtual camera and queues a preview frame for the UI. This latest-frame policy limits backlog but does not itself measure frame age or guarantee a latency bound.

`modules/video_capture.py` opens OpenCV camera devices and measures a startup FPS estimate. `modules/face_analyser.py` provides detection paths and optional landmark completion. `modules/processors/frame/face_swapper.py` contains swap processing. `modules/virtual_camera.py` targets a fixed 1280x720, 30 FPS `pyvirtualcam` output. The README describes OBS virtual-camera setup. This is not evidence of a headless engine API, browser processing, WebRTC, native camera drivers or WhatsApp support.

## Revised target architecture

VSwap is web-first, but the web frontend does not become the inference engine. The intended future path is:

```text
Browser camera/file → explicit mode + consent → API/session authorization
  ├─ approved local browser CPU capability (unverified investigation)
  └─ WebRTC or asynchronous job → independent orchestration → reusable Python engine worker
                                               └─ provider adapter → external GPU (vendor TBD)
```

WebRTC is reserved for persistent interactive media sessions; WebSocket handles signaling/control/progress, not per-frame media processing. Image/video work uses private media upload and asynchronous jobs. The current desktop pipeline remains an existing product adapter and engine reference; no implementation conversion is asserted.

## Boundaries and requirements

- Separate UI, API/control plane, signaling/media transport, orchestration, worker runtime, processing engine and provider adapter.
- Keep authentication, model/provider secrets, storage credentials and routing decisions out of the browser and engine package.
- Obtain explicit consent before camera acquisition and again before remote upload/streaming; distinguish local preview from transformed output and from a successful backend session.
- Bound queues and session/job resources; measure frame timestamps, capture-to-output age, FPS, jitter and dropped frames. Drop stale live frames instead of accumulating delay.
- On camera denial, unsupported local runtime, worker/provider failure, overload or network loss, show a truthful recoverable state; never present an untransformed preview as a successful swap.
- Avoid raw media and face embeddings in logs/telemetry; establish storage, retention and deletion controls before cloud processing.

## Status

Architecture proposal only. The local desktop flow and related modules are present, but this assessment did not run a camera, GPU benchmark, cloud worker or WebRTC session. Refer to [web platform architecture](WEB_PLATFORM_ARCHITECTURE.md), [WebRTC design](WEBRTC_STREAMING_ARCHITECTURE.md), [worker design](CLOUD_GPU_WORKER_ARCHITECTURE.md), and [performance targets](PERFORMANCE_AND_LATENCY_TARGETS.md).
