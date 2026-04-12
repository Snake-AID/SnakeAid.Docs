# LiveKit Client Contract Notes

This analysis file preserves the client-facing integration notes that originally lived in the standalone `live-kit-cloud.usageguide.md`.

## Stable contract carried forward into consultation docs

- `POST /api/consultations/{consultationId}/video-token`
  - returns consultation-scoped `token`, `wsUrl`, `roomName`
- `POST /api/videocall/livekit-token/demo/{roomname}`
  - development-only token generation for connectivity testing
- `POST /api/videocall/livekit-webhook`
  - provider callback endpoint

## Client-relevant rules

- `roomName` follows `consultation-{consultationId}` for production consultation sessions
- experts receive screen-share publish grants; other participants do not
- LiveKit credentials stay server-side only
- Flutter client flow remains:
  1. request backend token
  2. extract `token` and `wsUrl`
  3. connect with `livekit_client`

## Historical note

The standalone LiveKit usage guide was merged into this analysis record on 2026-04-12. The authoritative client contract for consultation remains under `consultation.usageguide*.md`.
