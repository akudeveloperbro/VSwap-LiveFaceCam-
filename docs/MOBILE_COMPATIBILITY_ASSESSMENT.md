# Mobile compatibility assessment

## Status

No Android/iOS application, system camera extension, mobile build configuration, or mobile compatibility tests were found in the inspected repository. Mobile WhatsApp compatibility is therefore unknown and is not part of verified support. A desktop `pyvirtualcam` integration should not be assumed to work on phones.

## Questions to research separately

For each OS and OS version, determine whether a third-party app can provide transformed frames as a system camera to another app, using a documented and distributable platform API. Verify background execution, camera ownership, permission prompts, thermal limits, app-store policy, extension signing/provisioning, and whether WhatsApp exposes the selected camera source to calls. Distinguish native app support from browser-based WhatsApp support.

### Android investigation

- Identify documented camera-provider/virtual-device options available to ordinary installed apps on the target Android versions; do not assume an app can register a system-wide camera.
- Test GPU/runtime availability, sustained thermal behavior, camera concurrency, permission revocation, screen lock/background transitions and battery impact.
- If system-camera exposure is unavailable, evaluate a supported in-app video-call workflow only as a separate product direction; it would not be WhatsApp integration.

### iOS investigation

- Determine whether supported camera extension or broadcast mechanisms can supply live transformed camera frames to WhatsApp calls. Do not confuse screen broadcasting with a selectable camera source.
- Verify extension entitlement/approval, user consent, lifecycle, latency, thermal and App Store distribution restrictions using official platform documentation and a device prototype.
- If the OS/client does not expose a supported path, mark direct WhatsApp camera compatibility infeasible rather than proposing private APIs or jailbreak techniques.

## Decision gates

1. Produce a platform capability note with authoritative API references and target OS/client versions.
2. Build a minimal, non-production prototype only after the supported mechanism is established.
3. Measure sustained 720p30, p95 frame age, frame drops, thermal/battery use and recovery on named devices.
4. Run an authorized WhatsApp test call; camera enumeration alone is insufficient.
5. Obtain product and security approval before committing to a mobile app or materially different architecture.

## Safe alternatives

Until a supported, verified mechanism exists, present mobile direct-camera integration as unsupported/unknown. A mobile companion may still manage account settings or transfer non-sensitive configuration if separately designed, but must not claim it can replace WhatsApp's camera feed.
