---
doc_role: baseline
module: refresh-token-per-devices
kind: layer
status: active
last_updated: 2026-04-12
owners: [backend-team]
---

# Refresh Token Per Devices - Introduction

## Overview
This layer defines how SnakeAid issues, rotates, and revokes refresh tokens per login session instead of per user.

The goal is to support multiple concurrent devices without cross-device invalidation:
- Login on device B must not revoke refresh capability on device A.
- Refresh on one device must not break refresh on another active device.
- Logout must revoke only the current session when the access token carries a session identifier.

## Why This Layer Exists
The previous implementation stored one refresh token and one refresh-token expiry per user in `AspNetIdentity.UserTokens`.
That model was simple but had two hard constraints:
- only one active refresh token per user
- logout and token rotation were effectively user-wide

The current layer moves refresh-token state into a dedicated session store so the backend can track refresh lifecycle per login session.

## Scope
This layer covers:
- session-scoped refresh token persistence
- refresh token rotation rules
- current-session logout behavior
- legacy fallback from old `AspNetIdentity.UserTokens` refresh tokens
- claims required in JWT access tokens to identify a login session

This layer does not cover:
- password policy
- OTP issuance and verification behavior
- Google OAuth token validation details
- access-token blacklist or immediate access-token revocation

## Core Business Rules
- Each successful login-style flow creates a distinct auth session.
- Each auth session owns exactly one active refresh token at a time.
- Refresh rotates the token for that session only.
- A rotated refresh token invalidates only the previous token of the same session.
- Logout revokes only the current session when the request is authenticated by an access token containing `sid`.
- Access tokens remain valid until expiry; logout revokes refresh, not already-issued access tokens.

## Backward Compatibility
- `POST /api/auth/refresh` request contract is unchanged: client still sends `userId` and `refreshToken`.
- Existing legacy refresh tokens stored in `AspNetIdentity.UserTokens` remain refreshable.
- On successful refresh through the legacy path, the backend issues a new session-based refresh token and removes the legacy token pair.

## Security Characteristics
- Refresh tokens are stored as SHA-256 hashes in the session table.
- Session lookup is performed by hashed refresh token, not by plaintext persistence.
- Session refresh uses optimistic concurrency on the hashed token field to prevent double-rotation races from succeeding.

## Non-Functional Expectations
- Safe for concurrent refresh attempts against the same session.
- Safe for multi-device login and refresh flows.
- Minimal rollout risk because API request contract remains stable for mobile clients.
