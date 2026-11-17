# Performance and latency targets

> **Architecture revision:** These targets were originally framed for local desktop processing. They are not verified web, browser-CPU, WebRTC, or cloud-worker baselines. See [web platform architecture](WEB_PLATFORM_ARCHITECTURE.md), [hybrid processing](HYBRID_PROCESSING_ARCHITECTURE.md), and [WebRTC architecture](WEBRTC_STREAMING_ARCHITECTURE.md) for the proposed web direction.

## Goal and baseline status

Priority order: stable real-time operation, visual quality, then client compatibility. The user-specified initial target is 720p output at 30 FPS; 60 FPS is an optimization target, not a launch promise. This audit did not run benchmarks or attach a camera/GPU. README and `ROADMAP.md` quote performance observations from prior project work, but they are not a reproducible current baseline for this branch or a WhatsApp call.

`benchmark_pipeline.py` is a local webcam benchmark, currently configured for 1080p60, a 200-frame run, and the InSwapper model. It times queue wait, detection, swap and paste-back; it does not establish end-to-end WhatsApp output latency, full stage timing for the current HyperSwap default, or representative hardware/network results. Treat existing headline FPS claims as unverified until reproduced with a documented environment.

## Metrics to implement

Use monotonic timestamps and aggregate measurements, not captured media or face identity data.

| Metric | Definition |
|---|---|
| Input FPS | Frames captured / elapsed time over a stated window. |
| Output FPS | Frames submitted successfully to output / elapsed time. |
| Capture-to-output age | Output submission timestamp minus capture timestamp for that frame; report p50/p95 and dropped-frame counts. This is the primary local real-time latency measure. |
| Detection, landmarks, swap inference, compositing | Per-stage durations, separately sampled; report p50/p95 and sample counts. |
| Queue age / drops | Age on dequeue and number dropped at each bounded queue. Queue length alone is insufficient. |
| Render / conversion / output send | Time spent converting, pacing and submitting frames to the camera adapter. |
| GPU utilization and memory | Optional platform telemetry, labeled unavailable when provider/tooling cannot measure it. |
| Recovery | Time to resume healthy output after temporary camera/output/worker failure. |

Do not call capture-to-output “end-to-end call latency”: it excludes OS camera delivery, WhatsApp capture/encoding, network, remote decode/display and audio/video synchronization. Measure those separately only in an explicitly consented test call with a suitable instrumented remote endpoint.

## Acceptance targets (proposed, subject to reference-device approval)

- **720p30:** sustained output at a nominal 30 FPS on a documented reference device, with measured p95 capture-to-output frame age no greater than 100 ms during a 10-minute run; report output FPS, p95 age, drops, and memory trend. This is a proposed engineering gate, not current evidence.
- **Overload:** queues remain bounded; old frames are dropped in favor of recent frames; no steadily increasing frame age. Show an explicit degraded-performance status if the system cannot sustain the target.
- **60 FPS:** later optimization goal, requiring a separate device/model/output matrix and quality review. Do not lower resolution or model quality silently to claim it.
- **Long calls:** 30-minute soak on each launch reference configuration, no monotonic memory growth beyond an agreed tolerance, no stuck camera/model worker, and deterministic stop/restart.
- **Visual correctness:** no identity/landmark regressions, unacceptable flicker/ghosting, or stale transformed frames introduced to meet FPS. Track quality tests alongside performance.

## Benchmark protocol

1. Record OS/build, CPU, GPU/VRAM, driver, Python/runtime, ONNX execution provider, model name/version/license, camera model, requested and actual capture/output settings, and power/thermal mode.
2. Warm up model and camera; run a fixed authorized test scene and resolution for at least 10 minutes for 30 FPS acceptance. Repeat at least three times and report each run plus aggregate p50/p95; do not publish only the best run.
3. Measure camera input and output independently, including drops and capture-to-output age. Separate queue waiting from inference duration.
4. Repeat with default processing and each optional enhancer separately; record quality and memory trade-offs.
5. Test overload (slow inference), camera contention/disconnect, virtual-camera unavailability, and clean stop/restart.
6. Keep telemetry aggregate and local by default; never include media, embeddings, source paths, or credentials.

## Status

Targets are proposals; no comparable baseline or acceptance evidence was collected during this documentation audit.
