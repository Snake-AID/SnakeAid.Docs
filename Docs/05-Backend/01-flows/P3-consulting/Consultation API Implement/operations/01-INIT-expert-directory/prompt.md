---
doc_role: operation
operation_id: 01-INIT-expert-directory
generated_from: plan.md
status: done
created_at: 2026-03-05
---

# Prompt: Implement Expert Directory & Availability

## Requirements

Implement the API endpoints defined in the gap analysis of `plan.md`.

Specific tasks:

1. Create `ExpertsController` if it doesn't exist, properly attributed with `[ApiController]` and standard route prefix.
2. Implement `PUT me/settings` to update `ExpertProfile.ConsultationFee` and biography fields. Requires Expert-claim authorization.
3. Implement `POST me/time-slots/bulk` to ingest an array of time blocks and create corresponding `ExpertTimeSlot` records. Ensure duplicates or overlapping slots within the payload are handled cleanly.
4. Implement `GET /` to return a paginated list of verified experts with their specializations.
5. Implement `GET /{expertId}` to return a detailed profile.
6. Implement `GET /{expertId}/reviews` fetching from the `UserFeedback` table.
7. Implement `GET /{expertId}/time-slots` to return `Status == Available` and `StartTime > Now` slots for the given expert.
8. Keep mobile handoff limitations explicit in implementation notes:
   - Profile stats blocks are not covered in this operation.
   - `IsVerified` may remain provisional if no true verification source exists yet.
   - Expert settings still use one `ConsultationFee` only; do not introduce split pricing in Operation 01.

## Constraints

- Do not expose internal Domain Entities directly to the API responses. Use DTOs and mappers.
- Follow the existing multi-tier architecture in the SnakeAid backend (`Api` -> `Application` -> `Infrastructure`).
