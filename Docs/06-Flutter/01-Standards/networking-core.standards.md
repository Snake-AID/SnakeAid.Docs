# Networking Core Standards

This document defines the architectural rules for the core networking layer in SnakeAid.Mobile.

## 1. HttpService Responsibilities

- **Scope**: Serves as a general-purpose wrapper for HTTP requests (`GET`, `POST`, `PUT`, `DELETE`).
- **Timeouts**: Utilizes a standard 30-second connection and receive timeout suitable for mobile networks.
- **Interceptors**: Includes standard interceptors such as `LoggingInterceptor` and `TokenRefreshInterceptor`.
- **Error Normalization**: Responsibly normalizes `DioException` and backend `ValidationErrors` into user-friendly strings that can be directly displayed in the UI.

## 2. HealthCheckService Responsibilities

- **Scope**: Acts as a dedicated service for fast pre-flight connection testing.
- **Fail-Fast Mechanism**: Configured with a very strict 2-second timeout. It pings the `/health` backend endpoint.
- **Global Integration**: It is directly injected into `HttpService`. Before any `GET`, `POST`, `PUT`, or `DELETE` request is sent, a health check ping is executed. If the server is offline, it fails instantly with a timeout exception, sparing the user from waiting the standard 30-second limit of `HttpService`.

## 3. Configuration & Dependency Injection

- **Base URL**: Extracted from `.env` via `flutter_dotenv`, with a production fallback.
- **Providers**: Both `HttpService` and `HealthCheckService` are registered as root-level Riverpod providers in `http_provider.dart`. Consuming repositories (like `AuthRepository`) inject them via their constructor.

## 4. Error Normalization Contract

- Avoids duplicated `_handleError` functions across services/repositories.
- Standardizes timeout issues: `DioExceptionType.connectionTimeout` yields generic "Kết nối timeout" response.
- Exposes specific logic for 400, 401, 404, 409, 422, and 500 status codes with user-readable messaging.
