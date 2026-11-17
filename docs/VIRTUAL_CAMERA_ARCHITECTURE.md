# Virtual-camera architecture and compatibility boundaries

## Existing state (verified by source inspection)

`modules/virtual_camera.py` wraps `pyvirtualcam` and sends BGR frames at fixed 1280x720, 30 FPS, letterboxed to fit. The live processing worker opens it while its virtual-camera toggle is on and closes it when disabled and during worker cleanup. The module/README describe the OBS Virtual Camera device and setup. This establishes a local desktop adapter path in the repository, not a web output track, a native driver authored here, or compatibility with a named call client.

## Relationship to the web-first platform

The v0 web frontend and WebRTC Live Studio are separate from a system virtual camera. A browser can preview/process its own camera stream only where a verified local capability exists; a transformed track in one origin does not become a selectable system camera for WhatsApp Web or Desktop. The web architecture does not inject into other applications or bypass their camera selection/security boundaries.

For WhatsApp use, the current plausible desktop investigation remains a supported system virtual-camera device selected through the application's documented camera picker. Android/iOS would need distinct OS-supported native components and separate feasibility and distribution review. Do not claim WhatsApp compatibility until a real transformed output is seen by the remote participant in an authorized test call.

## Adapter boundary for the existing engine

Keep the engine output API separate from platform camera drivers:

```text
FrameProcessor → VideoOutput.send(frame, timestamp) → platform adapter
                                            ├─ existing pyvirtualcam / OBS route
                                            ├─ native Windows route (not implemented)
                                            ├─ native macOS route (not implemented)
                                            └─ mobile routes (unknown)
```

An adapter should expose supported dimensions, pixel formats, frame rates, startup failure, send/drop outcomes and deterministic close. Preserve the current adapter until any separately approved alternative passes the same tests. Do not allow platform driver state to leak into inference code.

## WhatsApp compatibility gates

Treat Desktop, Web, Android and iOS as distinct targets. For each row record exact client/build, OS, device/driver, camera selection, resolution/FPS, permission state, test date, sustained output, remote participant evidence, drops/latency, disconnect recovery and clean shutdown. Enumeration or local preview alone does not pass. Do not inject scripts, use hidden extensions/private APIs, intercept calls, or capture/route WhatsApp call audio.

## Open questions and status

No WhatsApp client/version or mobile path has been verified by this repository assessment. See [WhatsApp assessment](WHATSAPP_COMPATIBILITY_ASSESSMENT.md), [mobile assessment](MOBILE_COMPATIBILITY_ASSESSMENT.md), and [web roadmap](VSWAP_WEB_IMPLEMENTATION_ROADMAP.md).
