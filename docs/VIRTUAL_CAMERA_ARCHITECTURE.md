# Virtual-camera architecture

## Current state

`modules/virtual_camera.py` wraps `pyvirtualcam.Camera` and sends BGR frames at fixed 1280x720, 30 FPS, letterboxed to fit. The live processing worker opens it only while the virtual-camera toggle is on, closes it when disabled, and closes it in its `finally` block. The module's docstring and README describe using the OBS Virtual Camera device and require OBS Studio setup. `requirements.txt` includes `pyvirtualcam` only on Windows and macOS. These facts establish an adapter and an intended path, not verified WhatsApp compatibility, a repository-owned native driver, or a supported device on every OS.

## Adapter boundary

Keep the processing engine output API separate from platform camera drivers:

```text
FrameProcessor → VideoOutput.send(frame, timestamp) → platform adapter
                                            ├─ Windows virtual-camera adapter
                                            ├─ macOS virtual-camera adapter
                                            └─ Linux adapter (future, if in scope)
```

The interface should expose capabilities (supported dimensions, pixel formats, nominal frame rate), startup failure, send/drop outcomes, and deterministic close. Do not let platform-specific driver state leak into face-processing code. Maintain the current `pyvirtualcam` route as a compatibility adapter until a replacement has passed the same output tests.

## Platform investigation

### Windows

First validate the existing OBS/pyvirtualcam route on supported Windows versions and WhatsApp Desktop/Web. Record driver installation, camera enumeration, pixel format, requested versus actual dimensions/rate, multi-client behavior, uninstall/recovery and signing/distribution implications. A Media Foundation/DirectShow device driver should be a separate, approved research project; this repository currently contains no such driver implementation.

### macOS

Validate the existing output route and OS permissions on supported macOS versions and WhatsApp clients. CoreMediaIO camera extensions have packaging, entitlement, signing, user-consent and distribution requirements. Do not claim a native CoreMediaIO camera exists in this codebase; a native extension would be separately scoped and reviewed.

### Linux

No Linux virtual-camera driver is documented by the current adapter. If selected as a target, assess supported virtual-camera mechanisms and device permissions independently before planning implementation.

### Browser output

A browser page can process its own camera stream and render a transformed track for supported web APIs, but that does not provide that track to WhatsApp Web running on another origin. Do not use page injection, hidden extensions, or private APIs as an integration. For the initial path, expose a system virtual-camera device and select it through the supported browser camera picker.

### Mobile

Android/iOS system camera extension availability is unresolved. Do not assume a desktop virtual camera architecture transfers to mobile; see [mobile assessment](MOBILE_COMPATIBILITY_ASSESSMENT.md).

## Operational requirements

- Start only after explicit user opt-in; show virtual-camera state separately from local preview state.
- Report device/driver unavailable and camera-busy errors without crashing or repeatedly reopening on every frame.
- Handle unsupported format, send failure, disable, window close, process exit and app restart; always close resources.
- Keep a bounded output queue and timestamp frames; count dropped frames and report output cadence. Never allow stale-frame backlog to grow.
- Do not capture or route microphone audio. Audio remains selected and routed by the call client.
- Avoid storing frames or logging image bytes. Document driver install/removal and permissions before distributing binaries.

## Verification gates

1. Adapter unit tests cover format conversion/letterboxing, startup failure and idempotent close.
2. Hardware smoke test verifies the device appears in supported OS camera enumeration and receives changing processed frames.
3. WhatsApp end-to-end call test verifies the remote participant sees transformed output (not merely the app preview).
4. Long-session test records frame pacing, drops, output age, memory growth, camera disconnect/recovery, and clean shutdown.

No platform/device has been verified by this assessment.
