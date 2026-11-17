# WebRTC Streaming Architecture

**Status:** Proposed; no browser streaming, signaling or WebRTC service was found in the inspected repository. Updated 2026-11-18.

## Purpose and separation

Use WebRTC for interactive live audio/video transport between a browser Live Studio and an authorized VSwap streaming endpoint. Use HTTPS for session creation/configuration and WebSocket for signaling and coarse state/events where appropriate. Do not use REST requests per video frame. WebSocket is not a substitute for media congestion control, codec negotiation, jitter buffers or peer connection lifecycle.

WebRTC architecture does not establish WhatsApp compatibility. A transformed track in one browser application cannot be injected into WhatsApp Web on another origin. Desktop virtual-camera integration and mobile system-camera integration are separate native/OS workstreams; see `VIRTUAL_CAMERA_ARCHITECTURE.md`, `WHATSAPP_COMPATIBILITY_ASSESSMENT.md`, and `MOBILE_COMPATIBILITY_ASSESSMENT.md`.

## Candidate topology

```text
Browser getUserMedia (explicit permission)
   │ local preview + negotiated WebRTC media
   ├─ HTTPS session API / WSS signaling → independent control plane
   └─ DTLS-SRTP media → approved WebRTC gateway/session endpoint → pinned GPU worker
                                                   │ transformed video return track
Browser Live Studio ←─────────────────────────────┘
```

The actual media termination location, SFU/media server versus direct worker endpoint, TURN topology and signaling protocol remain architecture decisions. Do not expose a raw worker address without authenticated admission and abuse controls. TURN may be required for restrictive networks; relay cost and privacy implications must be measured.

## Session flow

1. Explain local versus remote processing, consent, retention and any cost/usage impact; obtain explicit start action and camera permission.
2. Request a backend session with operation, approved mode, bounded media configuration and policy consent. Backend validates account, entitlement, rate limits, region and worker capacity.
3. Issue a short-lived, single-purpose signaling/media credential. Exchange offer/answer and ICE candidates through authenticated signaling; validate origin, session binding, message size/rate and expiration.
4. Admit session to a worker; report `starting` until worker and media path are actually ready. Keep camera preview and transformed output states distinct.
5. Return transformed video; report measured FPS/frame age/jitter/drops with measurement definitions. Do not fabricate metrics when unavailable.
6. On stop, permission revocation, tab close, timeout or failure: stop local tracks, close peer connection, invalidate credentials, end worker session, clear ephemeral media and emit terminal status.

## Real-time requirements

- One persistent session per active stream; bounded queues, stale-frame drop policy and explicit overload/degraded state.
- Codec/resolution/framerate negotiation with allowlists; adapt only through a visible, policy-defined profile and never silently misrepresent output quality.
- Instrument capture, encode, transport, queue, inference, decode/render timestamps where available. Report end-to-end app frame age separately from network RTT and avoid calling it remote call latency.
- Handle ICE restart/reconnect with bounded attempts and a visible reconnecting state; prevent duplicate worker sessions on retries.
- Separate signaling authorization from media-plane authentication; short-lived credentials and replay protection are required.
- Keep audio disabled unless a separately approved product requirement and consent model exists. The VSwap architecture does not capture or route WhatsApp call audio.

## Security and privacy

WebRTC encrypts transport but does not itself provide application authorization, consent, storage controls, content policy or tenant isolation. Enforce authenticated session binding, origin/CORS policy, rate/concurrency limits, abuse controls, resource budgets and region restrictions. Avoid recording by default. If recording or server-side persistence is later approved, document purpose, consent, retention, deletion and access controls before enabling it.

## Validation gates

Test permission denial/revocation, unsupported codecs, no TURN route, packet loss/jitter, bandwidth changes, worker overload/crash, signaling reconnect, duplicate start, session expiry and deterministic teardown. Measure p50/p95 capture-to-return age, FPS, jitter, dropped frames, worker readiness and network conditions on named browsers/devices. A camera preview or successful peer connection is not proof of inference or a valid transformed output.
