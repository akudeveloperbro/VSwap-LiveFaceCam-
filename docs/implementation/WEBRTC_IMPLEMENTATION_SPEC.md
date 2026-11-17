# WebRTC Implementation Specification

**Status:** Proposed; no WebRTC service exists in repository. Refer to [`../WEBRTC_STREAMING_ARCHITECTURE.md`](../WEBRTC_STREAMING_ARCHITECTURE.md).

## Transport split

HTTPS creates/authorizes sessions and returns a short-lived signaling credential. Authenticated WebSocket may carry SDP offer/answer, ICE candidates, session control and coarse state. WebRTC transports media using DTLS-SRTP, ICE and negotiated codecs; TURN relay may be required and must be evaluated for cost/privacy. Exact media server/SFU/direct worker topology is unresolved. Do not expose GPU workers directly or send frames via REST.

## Proposed session state

`idle → permission_requested → local_preview → admission_pending → signaling → worker_starting → connected → transformed → degraded/reconnecting → stopping → ended`; any stage may enter a typed failure state. The API and client distinguish local preview, connected media, actual transformed frames, and degraded output. Session ID is opaque; all signaling is bound to authenticated user, origin and expiry.

## Signaling contract (proposal)

Envelope `{type, session_id, sequence, sent_at, payload}`. Types: `offer`, `answer`, `ice_candidate`, `restart`, `close`, `status`, `error`. Validate allowed SDP/ICE shape, message size, origin, sequence, rate, session and TTL. Do not log SDP candidates or credentials by default. Negotiate approved camera profiles/codec set; no hidden or unlimited resolution/framerate. Server events include event ID, sequence and timestamp; clients refetch authoritative status after reconnect.

## Media/session behavior

Acquire camera only after clear user action and browser permission; display local preview separately. Request per-session remote consent before connecting cloud. Start worker only after admission succeeds. Preserve capture timestamps through processing; measure camera capture-to-return age separately from network RTT. Use bounded queues and drop stale frames, maintain pacing and return explicit degraded/overload state. Any network or worker switch requires visible reconnection/session re-negotiation; no silent cloud fallback. Do not capture call audio in initial scope.

Stop must close MediaStream tracks, peer connection, WebSocket, invalidate credentials, release worker lease, clear buffers and publish terminal state even on partial start. Handle tab close, permission revocation, ICE failure, TURN absence, background throttling, network change, app sleep and duplicate start.

## Acceptance gates

Run supported desktop/mobile browser matrix with network shaping: loss, jitter, bandwidth reduction and disconnect. Verify permission denied/revoked, unsupported codec, no route, signaling replay/rate abuse, worker overload/crash, reconnect, duplicate stop and cleanup. Measure end-to-end application frame age p50/p95, output FPS, jitter, drops, worker start/model load, network RTT and cost separately. Proposed 720p30 and p95 frame age target are specified in the performance doc; test on approved hardware and do not call it WhatsApp latency. A remote participant seeing transformed output is required for compatibility claims.
