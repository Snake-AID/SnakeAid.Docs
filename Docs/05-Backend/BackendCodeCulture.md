# Backend Code Culture (Dominant)

## Purpose

This document defines the dominant backend coding culture used in `SnakeAid.Backend`.
It acts as a practical AGENTS-style standard for implementation and refactoring decisions.

## 1) Architectural Boundaries

- Keep clean layering:
  - `Api` = transport concerns (HTTP, auth attributes, route contracts, response wrapping).
  - `Service` = business logic and domain rules.
  - `Repository` = data access only.
  - `Core` = domains, DTOs, exceptions, shared contracts.
- Do not move business decisions into controllers.
- Controllers should orchestrate calls, not implement workflows.

## 2) Response Contract (Dominant)

- Success responses should use `ApiResponseBuilder` and return `ApiResponse<T>`.
- Error responses should be produced by throwing typed exceptions and letting middleware format them.
- Avoid ad-hoc anonymous response shapes like:
  - `new { success = true, ... }`
  - `new { message = "..." }`
  - `new { error = "..." }`
- Canonical error formatter: `ApiExceptionHandlerMiddleware`.

## 3) Exception Strategy

- Throw domain-specific exceptions from service/controller guard clauses:
  - `BadRequestException`, `ValidationException`, `UnauthorizedException`,
  - `ForbiddenException`, `NotFoundException`, `ConflictException`, `BusinessException`.
- Do not map all persistence errors to `409`.
- Keep specific handling for concurrency (`DbUpdateConcurrencyException -> ConflictException`).
- For non-concurrency `DbUpdateException`, log with full exception and rethrow.
- Do not swallow cancellation:
  - `OperationCanceledException` must bubble up.

## 4) Controller Style

- Prefer inheriting `BaseController<T>` when claim helpers are needed.
- Keep controller actions thin:
  - extract `userId`/claims,
  - call service,
  - return wrapped success response.
- Avoid broad `catch (Exception)` that returns handcrafted `500` bodies.
- Let middleware handle exceptions whenever possible.

## 5) Validation Style

- Request DTOs use DataAnnotations as first layer.
- Value-type fields that must be required from payload should be nullable (`DateTime?`, `TimeSpan?`, `decimal?`) + `[Required]`.
- Complex rule validation belongs in `IValidatableObject` and/or service-level validation.
- If model validation is used in a controller area, keep it consistent (`[ValidateModel]` where that pattern is applied).

## 6) Logging Style

- Use structured logging with named placeholders.
- Log at correct severity:
  - `Information` for normal flow milestones.
  - `Warning` for recoverable conflicts/business failures.
  - `Error` for unexpected technical failures.
- Do not hide root exception details when debugging persistence/runtime issues.

## 7) Time and Date Policy

- Persist and process time in UTC.
- Validate timezone assumptions at API boundary.
- Never rely on server local timezone for domain logic.

## 8) Testing Culture

- Unit tests for business branching and exception mapping.
- Integration tests for runtime behavior that depends on relational constraints/transactions.
- Do not rely on EF InMemory for cases requiring FK/unique/index/transaction semantics.

## 9) Operation Refactoring Rule

When standardizing an existing operation:

1. Preserve endpoint purpose and authorization.
2. Align success/error contracts to this culture.
3. Update tests immediately for changed response wrappers.
4. Re-run affected test suites before closing.

## 10) Practical Decision Heuristic

- If a change improves consistency with `ApiResponseBuilder + ApiException middleware`, prefer it.
- If consistency conflicts with published client contract, document it and phase migration explicitly.

## 11) Multi-Replica Background Job Rule

- Any background job that changes domain state or moves money must be treated as a distributed coordination problem when multiple API replicas can run at the same time.
- Do not assume `IHostedService` is singleton at deployment level. It is singleton only inside one process.
- If the application can scale horizontally, lifecycle jobs must use one of these patterns:
  - a dedicated single worker deployment
  - a centralized scheduler (`Hangfire`, `Quartz`, queue worker)
  - a distributed lock on the shared write database or shared infrastructure
- Idempotency checks alone are not enough when two replicas can read the same eligible rows before either commit completes.
- For financial side effects such as payout, refund, or escrow settlement:
  - coordinate execution across replicas
  - keep state transition and money movement inside one DB transaction when possible
  - use row-level/pessimistic locking, concurrency tokens, or transaction-scoped distributed locking on the same aggregate
- If a hot patch is needed before a larger redesign, prefer a database-backed distributed lock that matches the current write topology and document that it is an interim operational safeguard, not the final scheduling architecture.
