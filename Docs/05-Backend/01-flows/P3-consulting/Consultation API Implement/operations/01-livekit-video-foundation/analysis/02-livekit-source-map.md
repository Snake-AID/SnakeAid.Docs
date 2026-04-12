# LiveKit Source Map

This analysis file preserves the original implementation map for the LiveKit integration that later became part of the consultation baseline.

## Main backend pieces

- `VideoCallController`
  - consultation-scoped token endpoint
  - demo token endpoint
  - LiveKit webhook endpoint
- `ILiveKitService` / `LiveKitService`
  - generate access tokens
  - call provider room APIs
  - validate webhook signatures
- `ILiveKitApi`
  - Refit client for Twirp room operations
- `LiveKitOptions`
  - `ApiKey`
  - `ApiSecret`
  - `WsUrl`
  - `TokenTtlMinutes`
  - `RoomEmptyTimeoutSeconds`

## DTO families introduced

- requests:
  - `VideoGrants`
  - `CreateRoomRequest`
  - `DeleteRoomRequest`
  - `LiveKitWebhookPayload`
- responses:
  - `VideoTokenResponse`
  - `RoomInfoResponse`
  - `ListRoomsResponse`

## Integration rules captured in the original standalone docs

- provider HTTP base URL is derived from `WsUrl`
- role-based publish sources are encoded into the JWT grant
- webhook validation uses provider-signed authorization headers
- provider-specific room cleanup later became an input to the consultation lifecycle worker

## Historical note

The standalone `live-kit-cloud.sourcecode.md` file was merged into this analysis note on 2026-04-12. The authoritative current-state description now lives in `consultation.sourcecode.md`.
