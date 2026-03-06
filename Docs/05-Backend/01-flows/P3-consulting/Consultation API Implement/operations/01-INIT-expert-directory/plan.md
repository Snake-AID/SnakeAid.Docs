---
doc_role: operation
operation_id: 01-INIT-expert-directory
type: INIT
status: done
created_at: 2026-03-05
affects:
  - Api/Controllers/ExpertController.cs
  - Service/Implements/ExpertService.cs
  - Service/Interfaces/IExpertService.cs
  - Core/Requests/Expert/ExpertSettingsRequest.cs
  - Core/Requests/Expert/BulkTimeSlotRequest.cs
  - Core/Responses/Expert/ExpertProfileResponse.cs
  - Core/Responses/Expert/ExpertTimeSlotResponse.cs
  - Repository/Data/Configurations/ExpertTimeSlotConfiguration.cs
  - Repository/Migrations/20260305181346_AddUniqueExpertTimeSlotConstraint.cs
---

# Plan: Expert Directory & Availability

## 1. As-Is

The Consultation module currently lacks API endpoints. Entities for `ExpertProfile` and `ExpertTimeSlot` exist but are not exposed for configuration or public discovery via REST controllers.

## 2. Gap Analysis

- Missing API endpoints for experts to update their profile settings (fees, bio).
- Missing API for experts to automatically generate their discrete `ExpertTimeSlot` records week-by-week.
- Missing public facing API for users to browse, search, and view verified experts and their reviews.
- Missing API for users to see available bookable slots for a specific expert.

## 3. To-Be Design

Implement the following REST endpoints:

- `PUT /api/v1/experts/me/settings`
- `POST /api/v1/experts/me/time-slots/bulk`
- `GET /api/v1/experts`
- `GET /api/v1/experts/{expertId}`
- `GET /api/v1/experts/{expertId}/reviews`
- `GET /api/v1/experts/{expertId}/time-slots`

This creates the foundational functionality for the directory and expert scheduling.

## 4. Impacted Components

- **Controllers**: `ExpertsController`
- **Services**: `ExpertService`, `IExpertService`
- **DTOs**: `ExpertSettingsRequest`, `BulkTimeSlotRequest`, `ExpertProfileResponse`, `ExpertTimeSlotResponse`

## 5. Risks & Constraints

- Bulk time slot generation must gracefully handle overwrites, skipping, or rejecting duplicates if the expert submits the same block twice.
- Expert directory search should be paginated and potentially cached if load gets high.

## 6. Validation Plan

- Unit test for bulk time slot creation logic to ensure valid contiguous blocks.
- Manual API test (Swagger) to verify profile updates reflect correctly in the directory output.
