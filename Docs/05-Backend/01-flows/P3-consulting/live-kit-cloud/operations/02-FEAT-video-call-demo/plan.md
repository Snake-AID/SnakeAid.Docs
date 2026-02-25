---
doc_role: operation
operation_id: FEAT-video-call-demo
type: FEAT
status: draft
created_at: 2026-02-25
affects:
  - Controllers/LiveKitController
---

# FEAT-video-call-demo — Plan

## 1. As-Is (from sourcecode)

| Item          | Current                                                                  |
| ------------- | ------------------------------------------------------------------------ |
| Controller    | `LiveKitController`                                                      |
| Route prefix  | `api/livekit`                                                            |
| Endpoint 1    | `POST api/livekit/consultation/{consultationId}/video-token` [Authorize] |
| Endpoint 2    | `POST api/livekit/webhook` [AllowAnonymous]                              |
| Demo endpoint | ❌ does not exist                                                        |

## 2. Gap Analysis

1. **Route prefix**: `api/livekit` is implementation-specific. Should be domain-generic `api/videocall` since the controller represents "video call" as a feature, not "LiveKit" as a vendor.
2. **Endpoint naming**: Routes should include `livekit-` prefix to clarify provider context within the videocall domain.
3. **No demo endpoint**: The consultation endpoint requires a valid consultation + user ownership validation. Consultation module is not ready yet, blocking end-to-end testing.
4. **Need a demo endpoint**: A simplified token generator that accepts `{roomname}` directly, bypassing consultation validation, for development testing.

## 3. To-Be Design

| Item             | Target                                                                            |
| ---------------- | --------------------------------------------------------------------------------- |
| Controller       | `VideoCallController` (renamed from `LiveKitController`)                          |
| Route prefix     | `api/videocall`                                                                   |
| Endpoint 1       | `POST api/videocall/livekit-token/{consultationId}` [Authorize] — unchanged logic |
| Endpoint 2       | `POST api/videocall/livekit-webhook` [AllowAnonymous] — unchanged logic           |
| Endpoint 3 (NEW) | `POST api/videocall/livekit-token/demo/{roomname}` [Authorize] — dev/test only    |

### Demo endpoint behavior

- Takes `{roomname}` as a string path parameter
- Uses authenticated user's `userId` + `role` from JWT (still requires login)
- Grants same permissions as consultation endpoint (role-based)
- Bypasses consultation validation entirely
- Returns `{ token, wsUrl, roomName }` — same `VideoTokenResponse`

## 4. Impacted Components

| File                                            | Change                                                                                    |
| ----------------------------------------------- | ----------------------------------------------------------------------------------------- |
| `SnakeAid.Api/Controllers/LiveKitController.cs` | Rename to `VideoCallController.cs`, change route prefix, rename routes, add demo endpoint |

## 5. Risks & Constraints

- Demo endpoint should be clearly marked as dev-only (Swagger description)
- No breaking changes to service layer — only controller routes change
- If Flutter mobile is already using `api/livekit/...` routes, those calls will break → **Flutter has NOT been implemented yet**, so no risk

## 6. Validation Plan

- `dotnet build` — 0 errors
- Swagger UI — verify 3 endpoints under `VideoCall` tag
- Test demo endpoint via Swagger: login → authorize → call `/api/videocall/livekit-token/demo/test-room` → receive valid token
