# WhatsApp compatibility assessment

## Purpose and status

This is an architecture assessment, not a compatibility claim. The repository is currently a local Python/PySide6 desktop application (see `README.md`, `run.py`, `modules/ui.py`); it does not contain a web dashboard, browser extension, WebRTC service, or WhatsApp integration. No WhatsApp client/version/OS test has been run as part of this assessment.

## Paths considered

| Target | Plausible integration path | Current evidence | Assessment |
|---|---|---|---|
| WhatsApp Desktop | Select a system virtual camera as the call's camera. | `modules/virtual_camera.py` sends rendered BGR frames to `pyvirtualcam`; the README describes the OBS Virtual Camera device. | Most plausible first target, but device visibility, call selection, permissions and sustained-call stability must be verified on each supported OS/client. No WhatsApp-specific code is needed if its camera selector accepts the device. |
| WhatsApp Web | Select a system virtual camera exposed to the browser, then grant camera permission to WhatsApp Web. | The repository has no browser app, browser media pipeline, extension, or transformed `MediaStreamTrack`. | A desktop virtual camera is the safe initial route. Do not inject scripts into WhatsApp, intercept its internal APIs, or bypass browser/client security. A web dashboard cannot silently replace another origin's camera track. |
| Android/iOS WhatsApp | Supply a system camera device or supported OS camera extension. | No mobile code, OS extension, or mobile test harness exists. | Unknown and likely platform-constrained; investigate separately. Do not promise mobile compatibility. |

WhatsApp client behavior and OS virtual-camera support are external prerequisites and may change. Verify through supported camera selection and documented OS APIs; do not rely on undocumented hooks.

## Compatibility test matrix

Before publishing support, record the exact WhatsApp client/build, OS version, camera device/driver, GPU/runtime, camera resolution/FPS, and test date. For each configuration verify:

1. The virtual camera appears in WhatsApp's camera selector and can be selected after app restart.
2. A consenting test subject produces transformed frames in a test call; the local preview alone is not evidence of call output.
3. Input/output FPS, end-to-end video age/latency, visual quality, dropped frames and thermal/memory behavior remain acceptable over a sustained call.
4. Camera permission denial, another app owning the camera, device disconnect, worker failure, app restart and virtual-camera shutdown recover safely.
5. WhatsApp receives audio directly from the user's separately selected microphone; this application does not capture, relay, process or claim to preserve call audio.

## Decisions and guardrails

- Prioritise a native desktop virtual-camera path for the first compatibility spike; keep WhatsApp clients unmodified.
- Treat WhatsApp Desktop and WhatsApp Web as distinct test targets even if they share an OS camera device.
- The application processes video only. It must not inspect, modify, or bypass WhatsApp encrypted communications.
- Do not list a client as supported until the exact matrix row passes an end-to-end call test.
- See [virtual-camera architecture](VIRTUAL_CAMERA_ARCHITECTURE.md), [pipeline](REALTIME_VIDEO_PIPELINE.md), and [implementation plan](REALTIME_ENGINE_IMPLEMENTATION_PLAN.md).

## Open questions

- Which OS/client versions and hardware are launch targets?
- Is the intended product a local desktop tool with an optional web control plane, or a hosted processing service? The former fits this repository and avoids remote-camera transport; the latter changes privacy, bandwidth, auth, storage, and operational requirements substantially.
- What is the acceptable p95 camera-to-output latency on the reference device?
