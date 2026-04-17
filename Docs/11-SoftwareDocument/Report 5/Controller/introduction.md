# Consultation Unit Test Report Expansion Plan

## Objective

Add the missing Consultation flow into the unit test report pipeline for [SnakeAid Report5_Unit Test.xlsx](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report 5/SnakeAid Report5_Unit Test.xlsx), using Markdown as the only editing surface until the Consultation section is complete and ready for workbook write-back.

## Current State

- The workbook currently contains matrix content for two flows only:
  - SnakeBite Incident
  - Snake Catching
- The Markdown working copy already mirrors workbook sheets `01` to `20` under:
  - `D:\SourceCode\Snake_AID\SnakeAid.Docs\Docs\11-SoftwareDocument\Report 5\SnakeAid Report5_Unit Test_markdown`
- Consultation already has distilled source-of-truth documentation under:
  - `D:\SourceCode\Snake_AID\SnakeAid.Docs\Docs\05-Backend\01-flows\P3-consulting\DistiledDocs`
- The backend codebase already implements Consultation controllers, services, route conventions, and multiple unit/integration tests that can be used to derive matrix conditions and confirmations.

## Verified Knowledge Sources

Primary sources to use while building Consultation matrices:

1. `consultation.usageguide.md`
2. `consultation.sourcecode.md`
3. `consultation.introduction.md`
4. Existing report matrix patterns in:
   - `07-getincidentdetail.md`
   - `16-getsnakecatchingrequestdetail.md`
   - `18-assignsnakecatchingrequest.md`
   - `20-example.md`
5. Consultation-related tests in `SnakeAid.Backend`, especially:
   - `ConsultationRouteConventionTests.cs`
   - `ConsultationBookingsControllerIntegrationTests.cs`
   - `ConsultationPaymentIntegrationTests.cs`
   - `ScheduledConsultationIntegrationTests.cs`
   - `EmergencyConsultationIntegrationTests.cs`
   - `AdminConsultationHistoryIntegrationTests.cs`
   - `ExpertControllerIntegrationTests.cs`
   - `ConsultationPropertyTests.md`

## Recommended Scope

### Core report scope to include now

These are the Consultation functions that belong to the main business flow and should be added as unit test matrix sheets:

1. `GetExperts`
2. `GetExpertProfile`
3. `GetExpertReviews`
4. `GetExpertTimeSlots`
5. `UpdateExpertSettings`
6. `CreateBulkTimeSlots`
7. `GetExpertConsultations`
8. `CreateConsultationBooking`
9. `GetMyScheduledBookings`
10. `GetExpertScheduledBookings`
11. `PayScheduledBooking`
12. `CreateEmergencyConsultationRequest`
13. `PayEmergencyRequest`
14. `AcceptEmergencyConsultationRequest`
15. `RejectEmergencyConsultationRequest`
16. `ConfirmConsultationPayment`
17. `GetMyConsultations`
18. `GenerateVideoToken`
19. `EndConsultation`
20. `CreateConsultationReview`
21. `GetConsultationReview`
22. `GetAllConsultations`
23. `GetConsultationById`

### Supporting surfaces to keep out of the first workbook pass

These should stay out of the first Consultation report pass unless explicitly requested:

- `POST /api/media/upload-image`
- `GET /api/snake-species/search`
- LiveKit demo token endpoint
- LiveKit webhook endpoint
- SignalR hubs as standalone sheets

Reason:
The current report workbook is function-sheet oriented around main business APIs. The surfaces above are auxiliary, infrastructure-facing, or better represented as supporting conditions inside relevant Consultation sheets rather than separate workbook functions.

## Workbook Integration Strategy

### Markdown-first rule

Do not edit the workbook directly while the Consultation section is incomplete.

Execution order:

1. Extend the Markdown working set first.
2. Add Consultation function rows into the Markdown version of `03-functions.md`.
3. Create new numbered Markdown sheet files for Consultation functions.
4. Normalize each sheet to the same matrix shape already used in SnakeBite Incident and Snake Catching.
5. Review numbering, sheet naming, and cross-reference consistency.
6. Only after all Consultation Markdown sheets are complete, transfer them into the workbook.

### Proposed sheet numbering

The current Markdown workbook mirror ends at `20`.
Recommended Consultation sheets should begin at `21`.

Suggested naming pattern:

- `21-getexperts.md`
- `22-getexpertprofile.md`
- `23-getexpertreviews.md`
- `24-getexperttimeslots.md`
- `25-updateexpertsettings.md`
- `26-createbulktimeslots.md`
- `27-getexpertconsultations.md`
- `28-createconsultationbooking.md`
- `29-getmyscheduledbookings.md`
- `30-getexpertscheduledbookings.md`
- `31-payscheduledbooking.md`
- `32-createemergencyconsultationrequest.md`
- `33-payemergencyrequest.md`
- `34-acceptemergencyconsultationrequest.md`
- `35-rejectemergencyconsultationrequest.md`
- `36-confirmconsultationpayment.md`
- `37-getmyconsultations.md`
- `38-generatevideotoken.md`
- `39-endconsultation.md`
- `40-createconsultationreview.md`
- `41-getconsultationreview.md`
- `42-getallconsultations.md`
- `43-getconsultationbyid.md`

## Matrix Construction Rules

For each Consultation sheet:

- Keep the same header structure used by existing report Markdown sheets.
- Define clear preconditions first.
- Separate input conditions from confirmation rows.
- Map each UTCID column to exactly one meaningful test case combination.
- Prefer conditions that are code-verifiable from distilled docs or tests.
- Include HTTP status expectations when the function is API-facing.
- Include log/message expectations only where the current implementation or existing matrix style makes them stable enough to document.
- Mark test type as `N`, `A`, or `B` consistently.

### Matrix Body Formatting Rules

Non-negotiable formatting rule for workbook-mirror sheets:

- Each report sheet must be rendered as exactly 2 Markdown tables:
  - `Summary`: workbook rows `2` to `7`
  - `Matrix`: workbook rows `9` onward
- Do not keep one giant Markdown table that merges summary rows and matrix rows together.
- Row numbers are workbook context only and should not be kept as a leading `Row` column in the Markdown tables.
- Inside the matrix body, UTCID cells must use workbook-style marks only:
  - `O` for selected/applicable
  - blank for not selected
- Do not write prose sentences inside UTCID cells.
- Do not use narrative phrases like:
  - `Operation successful`
  - `validation error`
  - `sorted fee ascending`
  - `none`
- Put concrete values and expected outcomes on row labels only, following the existing workbook style.
- Prefer row patterns like:
  - condition name row
  - concrete value rows under that condition
  - confirm section rows such as `HTTP 200`, `HTTP 422`, exact log message text, `data = null`, `size > 0`
- If an outcome needs explanation, keep that explanation outside the matrix body in planning notes, not in the matrix cells.

In short:

- Matrix rows may contain labels and concrete values.
- Matrix UTCID intersections may contain only `O` or blank.

## Recommended Delivery Sequence

Build the Consultation report in the following batches:

### Batch 1: Expert discovery and setup

- GetExperts
- GetExpertProfile
- GetExpertReviews
- GetExpertTimeSlots
- UpdateExpertSettings
- CreateBulkTimeSlots
- GetExpertConsultations

### Batch 2: Scheduled consultation flow

- CreateConsultationBooking
- GetMyScheduledBookings
- GetExpertScheduledBookings
- PayScheduledBooking
- ConfirmConsultationPayment

### Batch 3: Emergency consultation flow

- CreateEmergencyConsultationRequest
- PayEmergencyRequest
- AcceptEmergencyConsultationRequest
- RejectEmergencyConsultationRequest

### Batch 4: Active consultation and review flow

- GetMyConsultations
- GenerateVideoToken
- EndConsultation
- CreateConsultationReview
- GetConsultationReview

### Batch 5: Admin monitoring

- GetAllConsultations
- GetConsultationById

## Known Risks

- Some existing sheet headers in the workbook still carry placeholder values such as `Function1` or mismatched function names; Consultation additions should avoid copying those mistakes forward.
- Not every Consultation behavior has a stable public log message suitable for matrix confirmation; response status and response shape should be preferred when messages are not contract-grade.
- SignalR behavior is part of the flow but not ideal as standalone unit test matrix sheets in the current workbook format.
- Existing workbook formulas in summary sheets may need adjustment after Consultation sheets are appended.

## Completion Definition

The Consultation report is ready for workbook write-back only when all of the following are true:

- `03-functions.md` contains the Consultation function list.
- All planned Consultation sheet Markdown files exist.
- Every Consultation sheet has explicit conditions, confirmations, UTCID columns, and result typing.
- Numbering order is stable and no filename conflicts remain.
- A final consistency pass confirms that the Markdown set can be transferred to the workbook without structural guessing.
