---
doc_role: baseline
module: refresh-token-per-devices
kind: layer
status: active
last_updated: 2026-04-12
owners: [backend-team]
---

# Refresh Token Per Devices - Usage Guide

This guide documents the client-visible behavior of the per-device refresh-token implementation in SnakeAid.

## Scope
This guide is limited to:
- session behavior for `login`
- session behavior for `refresh`
- session behavior for `logout`
- token storage expectations for multi-device clients

It does not repeat unrelated register, OTP, or profile details.

## What Changed For Clients
- Request contract for `POST /api/auth/refresh` is unchanged.
- Response shape for login and refresh is unchanged.
- Each successful login now creates a separate refresh-capable session.
- Refresh on one device does not revoke refresh on another device.
- Logout revokes the current session only when called with the current access token.

## AuthResponse Contract
Returned by:
- `POST /api/auth/login`
- `POST /api/auth/login/v2`
- `POST /api/auth/google`
- `POST /api/auth/register`
- `POST /api/auth/verify-account`
- `POST /api/auth/refresh`

Shape:

```json
{
  "accessToken": "<jwt_access_token>",
  "refreshToken": "<refresh_token>",
  "accessTokenExpiresAt": "2026-04-12T10:15:00Z",
  "refreshTokenExpiresAt": "2026-05-12T10:00:00Z",
  "user": {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "email": "user@example.com",
    "fullName": "John Doe",
    "avatarUrl": null,
    "role": "User",
    "isActive": true
  }
}
```

## Device/Session Rules For Clients
- Treat each login result as a distinct session.
- Persist the latest refresh token per device/app instance.
- Never reuse an old refresh token after a successful refresh response.
- Do not assume another device logout will revoke your current device.

## Endpoint Contracts

### 1. Login

**Endpoint**: `POST /api/auth/login`

**Request body**:

```json
{
  "email": "user@example.com",
  "password": "P@ssw0rd!"
}
```

**Behavior**:
- Creates a new auth session for the current device/app login.
- Returns a refresh token bound to that session only.
- Does not revoke previously active refresh tokens on other devices.

**Success response**:
- `200 OK`
- returns `AuthResponse`

### 2. Login V2

**Endpoint**: `POST /api/auth/login/v2`

**Request body**:

```json
{
  "email": "user@example.com",
  "password": "P@ssw0rd!",
  "role": "User"
}
```

**Behavior**:
- same session semantics as `POST /api/auth/login`
- role must match the account

### 3. Refresh

**Endpoint**: `POST /api/auth/refresh`

**Request body**:

```json
{
  "userId": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "refreshToken": "current_refresh_token"
}
```

**Behavior**:
- rotates the refresh token for the current session only
- returns a new access token and a new refresh token
- does not revoke refresh tokens issued to other active sessions/devices
- old refresh token for the same session becomes invalid immediately after success

**Success response**:
- `200 OK`
- returns `AuthResponse`

**Error responses**:
- `401 Unauthorized`: invalid, expired, revoked, or already-rotated refresh token
- `403 Forbidden`: account inactive

**Client requirement**:
- overwrite the locally stored refresh token immediately after each successful refresh

### 4. Logout

**Endpoint**: `POST /api/auth/logout`

**Auth**: required

**Request body**: none

**Required header**:

```http
Authorization: Bearer <access_token>
```

**Behavior**:
- revokes the session identified by the current access token
- does not revoke refresh tokens on other devices
- does not invalidate already-issued access tokens before their normal expiry

**Success response**:

```json
{
  "status_code": 200,
  "message": "Logged out successfully.",
  "is_success": true,
  "data": "Logged out successfully."
}
```

## Multi-Device Example

### Device A login
- receive `refreshToken_A`

### Device B login
- receive `refreshToken_B`

### Device B refresh
- send `refreshToken_B`
- receive `refreshToken_B2`
- `refreshToken_A` remains valid on Device A

### Device A logout
- call `POST /api/auth/logout` with Device A access token
- Device A refresh token is revoked
- Device B can still refresh with `refreshToken_B2`

## Legacy Token Compatibility
For clients still holding older refresh tokens issued before the session-based implementation:
- the first successful refresh can still work
- the backend migrates that refresh flow to the new session-based model
- the client does not need a request-format change for that migration

## Recommended Client Storage Model

### Mobile
- store the latest `accessToken`
- store the latest `refreshToken`
- overwrite both after every successful refresh
- clear only the current device storage on logout

### Web
- same token-rotation rule applies if tokens are managed by the frontend
- if multiple browser profiles/devices exist, each profile/device behaves as a separate session

## Failure Cases Clients Must Handle
- refresh token reused after success: expect `401`
- refresh token from a logged-out session: expect `401`
- refresh token expired: expect `401`
- account deactivated after login: refresh may return `403`

## Safe Client Flow
1. Login and store the returned token pair locally for that device.
2. Call protected APIs with the access token.
3. On access-token expiry, call `POST /api/auth/refresh`.
4. Replace the stored access token and refresh token with the new pair.
5. On logout, call `POST /api/auth/logout` and clear local tokens for that device only.
