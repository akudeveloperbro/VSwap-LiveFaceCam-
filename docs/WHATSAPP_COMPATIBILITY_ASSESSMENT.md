# WhatsApp compatibility assessment

## Product direction and status

VSwap's target product direction is now a web-first platform with a separate API/orchestration layer and reusable Python processing engine. This does not make WhatsApp compatibility automatic. The inspected repository remains a local Python/PySide6 desktop application with a `pyvirtualcam` adapter; it contains no web dashboard, browser extension, WebRTC service, or WhatsApp-specific integration. No WhatsApp client/version/OS end-to-end call test has been run as part of this documentation update.

## Compatibility paths (not claims)

| Target | Candidate supported route | Evidence/status | Additional components and gates |
|---|---|---|---|
| WhatsApp Desktop | Select a system virtual camera supplied by a local/native VSwap component. | Existing code wraps `pyvirtualcam`; README describes OBS Virtual Camera. Not verified with WhatsApp. | Local engine/runtime and OS virtual-camera support; device enumeration, permission and sustained remote-participant call tests per OS/client. |
| WhatsApp Web | Select an OS-level virtual camera in the supported browser camera picker. | Repository has no browser media pipeline or transformed `MediaStreamTrack`. A VSwap page cannot replace a camera track on another origin. | Same desktop virtual-camera path plus browser permissions/client test. Do not inject scripts or bypass origin boundaries. |
| Android/iOS | A documented OS-supported camera provider/extension, if available to third-party apps. | No mobile app, OS extension or test harness found; availability unknown. | Native components, OS entitlements/distribution review, thermal/latency tests, WhatsApp camera selection and end-to-end call evidence. Screen broadcast is not equivalent to a camera input. |

Browser-based Live Studio WebRTC streaming is a distinct VSwap product feature; it does not route the transformed stream into WhatsApp. Keep this separation explicit in product copy.

## Required test matrix

For every exact client/OS/hardware/driver combination, record versions and date. Verify: (1) virtual camera appears and remains selectable after restart, (2) consenting test subject yields transformed frames visible to a remote participant in a real test call, (3) sustained quality/FPS/frame age/drop/thermal/memory behavior, (4) camera permission denial, contention, disconnect, worker failure and recovery, and (5) deterministic shutdown. The app must not capture or route call audio. A local preview, successful camera enumeration, or browser WebRTC connection alone is not a pass.

## Safety and compatibility boundaries

Use supported camera selection and documented OS APIs only. Do not inject code into WhatsApp, intercept encrypted communications, use undocumented/private APIs or claim client compatibility based on another platform. Any native virtual camera, mobile component, OS signing/distribution, and support promise needs separate approval and validation.

## Related docs

- [Virtual camera architecture](VIRTUAL_CAMERA_ARCHITECTURE.md)
- [WebRTC streaming architecture](WEBRTC_STREAMING_ARCHITECTURE.md)
- [Mobile compatibility assessment](MOBILE_COMPATIBILITY_ASSESSMENT.md)
- [Web implementation roadmap](VSWAP_WEB_IMPLEMENTATION_ROADMAP.md)
