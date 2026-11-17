# AI Engine Integration Specification

**Status:** Proposed boundary and migration approach. Current repository has the local Python/PySide6 engine; it is not verified as a headless service. No engine source changes in this phase.

## Existing foundation (verified repository inspection)

Current modules include camera capture (`modules/video_capture.py`), face analysis (`modules/face_analyser.py`), processing (`modules/processors/frame/`), execution-provider selection (`modules/gpu_processing.py`/runtime code), UI orchestration (`modules/ui.py`), virtual camera (`modules/virtual_camera.py`), model loading/download support and image/video CLI paths. Existing desktop functionality/tests must remain intact. Current requirements include PySide6, OpenCV, InsightFace, ONNX Runtime variants, TensorFlow and pyvirtualcam; this dependency set is desktop-specific and not yet suitable as a web worker image. Current README and model credits list model licensing restrictions, including non-commercial/research conditions for some weights; perform legal/license review before deployment/distribution.

## Proposed modules and seams

- **Face detection:** detector interface returns bounded boxes/confidence and explicit coordinate space/version.
- **Recognition/embedding:** optional sensitive stage; keep data ephemeral unless a separately approved need and consent/retention design exist.
- **Tracking:** temporal association with confidence, expiry, re-detect and discontinuity reset; not verified in current live loop.
- **Alignment:** validated geometry/landmarks and model-specific transform.
- **Swapping:** model adapter with pinned model/runtime metadata and capability/error contract.
- **Blending/masking:** separate compositing strategy and tests for occlusion, edges and color behavior.
- **Enhancement:** optional, versioned stage that can be disabled and measured independently.
- **Frame pipeline:** bounded latest-frame queue, timestamps, cancellation and stage telemetry.
- **Inference runtime:** CPU/CUDA/DirectML/OpenVINO/CoreML capability adapter. Local desktop support does not establish cloud image or browser support.
- **Rendering/output:** frame encoder/return adapter separate from PySide UI, camera driver and WebRTC transport.

## Candidate engine protocol

A versioned Python package should expose `capabilities()`, `open_session(config)`, `process_frame(frame, capture_timestamp)`, `process_asset(input, options, progress_callback)`, `health()`, and idempotent `close_session()`. Define tensor/color/channel layout, pixel format, resolution/frame limits, timestamps/clock semantics, cancellation, error codes, version compatibility and model rights. Keep provider SDK, FastAPI, user database, storage credentials, billing and OS camera drivers out of the engine package.

## Extraction and test strategy

1. Map import graph, GUI/global state, model download assumptions, filesystem paths and locks.
2. Separate engine config/results from PySide types and UI callbacks via adapters without moving or renaming existing modules first.
3. Add engine unit tests with synthetic/generated test frames and model stubs; no real personal media in tests.
4. Compare old desktop flow and adapter flow for output equivalence/quality on license-approved fixtures; preserve old entrypoint.
5. Measure stage timing, p50/p95 frame age, drops, memory and model startup on a documented device/provider.
6. Test model unavailable/corrupt, unsupported runtime, cancellation, concurrent session isolation, worker crash and deterministic resource close.

Do not declare headless readiness until tests run without Qt, physical camera, user profile state or desktop-only dependencies. Do not redistribute weights until each license and source is approved.
