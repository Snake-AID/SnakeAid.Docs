# Consultation Unit Test Report Roadmap

## Purpose

Track the Markdown-first execution plan for adding the missing Consultation flow into the unit test report in a way that can be resumed at any time without guessing what was already finished.

## Working Rule

- This roadmap is the execution control file for the Consultation report expansion.
- Progress must be tracked by updating checkboxes in this file.
- Do not write to the workbook `.xlsx` until all required Markdown work is complete and checked off.
- If work is interrupted, resume from the first unchecked item in the current active phase.

## Scope Baseline

- Target workbook:
  - `D:\SourceCode\Snake_AID\SnakeAid.Docs\Docs\11-SoftwareDocument\Report 5\SnakeAid Report5_Unit Test.xlsx`
- Markdown working copy:
  - `D:\SourceCode\Snake_AID\SnakeAid.Docs\Docs\11-SoftwareDocument\Report 5\SnakeAid Report5_Unit Test_markdown`
- Planning workspace:
  - `D:\SourceCode\Snake_AID\SnakeAid.Docs\Docs\11-SoftwareDocument\Report 5\Controller`
- Consultation source docs:
  - `D:\SourceCode\Snake_AID\SnakeAid.Docs\Docs\05-Backend\01-flows\P3-consulting\DistiledDocs`

## Resume Protocol

When resuming:

1. Read `Controller/introduction.md`.
2. Read this file from top to bottom.
3. Find the first phase that is not fully checked.
4. Within that phase, continue from the first unchecked task.
5. Before moving to the next phase, confirm all exit checks in the current phase are completed.

## Current Status

- [x] Reviewed current report Markdown structure and matrix style.
- [x] Reviewed Consultation distilled documents.
- [x] Verified Consultation controller/API surface from backend source.
- [x] Reviewed Consultation-related tests for evidence sources.
- [x] Created planning documents in `Controller`.
- [x] Updated workbook-mirror metadata for Consultation.
- [x] Created Consultation Markdown sheet files.
- [x] Populated Consultation matrix content.
- [ ] Performed cross-sheet consistency pass.
- [ ] Wrote final Consultation section back into workbook.

## Confirmed First-Pass Scope

### Included function sheets

- [x] `GetExperts`
- [x] `GetExpertProfile`
- [x] `GetExpertReviews`
- [x] `GetExpertTimeSlots`
- [x] `UpdateExpertSettings`
- [x] `CreateBulkTimeSlots`
- [x] `GetExpertConsultations`
- [x] `CreateConsultationBooking`
- [x] `GetMyScheduledBookings`
- [x] `GetExpertScheduledBookings`
- [x] `PayScheduledBooking`
- [x] `CreateEmergencyConsultationRequest`
- [x] `PayEmergencyRequest`
- [x] `AcceptEmergencyConsultationRequest`
- [x] `RejectEmergencyConsultationRequest`
- [x] `ConfirmConsultationPayment`
- [x] `GetMyConsultations`
- [x] `GenerateVideoToken`
- [x] `EndConsultation`
- [x] `CreateConsultationReview`
- [x] `GetConsultationReview`
- [x] `GetAllConsultations`
- [x] `GetConsultationById`

### Explicitly excluded from first pass

- [x] `POST /api/media/upload-image`
- [x] `GET /api/snake-species/search`
- [x] LiveKit demo token endpoint
- [x] LiveKit webhook endpoint
- [x] SignalR hubs as standalone workbook sheets

## Phase 1. Freeze Naming, Numbering, and Metadata

Status: `Completed`

Goal:
Lock the workbook-mirror structure before authoring any Consultation matrix content.

### Phase 1 checklist

- [x] Confirm Consultation sheets start after current last sheet `20`.
- [x] Confirm final sheet numbering range is `21` to `43`.
- [x] Confirm final Markdown filenames match the agreed sheet names.
- [x] Update `03-functions.md` with Consultation function rows.
- [ ] Ensure each new row includes:
  - [x] requirement name
  - [x] class name
  - [x] function name
  - [x] sheet name
  - [x] short description
  - [x] pre-condition summary
- [x] Check whether `05-statistics.md` needs a note for future formula expansion.
- [x] Record any numbering or naming deviations in the Decision Log section below.

### Phase 1 deliverables

- [x] `03-functions.md` contains all Consultation function entries.
- [x] Consultation filename list is stable.

### Phase 1 exit check

- [x] No Consultation sheet creation starts before `03-functions.md` is updated.

## Phase 2. Create Empty Consultation Sheet Files

Status: `Completed`

Goal:
Create the full Markdown skeleton for every Consultation sheet so content work can resume file-by-file without ambiguity.

### Phase 2 checklist

- [x] Create `21-getexperts.md`
- [x] Create `22-getexpertprofile.md`
- [x] Create `23-getexpertreviews.md`
- [x] Create `24-getexperttimeslots.md`
- [x] Create `25-updateexpertsettings.md`
- [x] Create `26-createbulktimeslots.md`
- [x] Create `27-getexpertconsultations.md`
- [x] Create `28-createconsultationbooking.md`
- [x] Create `29-getmyscheduledbookings.md`
- [x] Create `30-getexpertscheduledbookings.md`
- [x] Create `31-payscheduledbooking.md`
- [x] Create `32-createemergencyconsultationrequest.md`
- [x] Create `33-payemergencyrequest.md`
- [x] Create `34-acceptemergencyconsultationrequest.md`
- [x] Create `35-rejectemergencyconsultationrequest.md`
- [x] Create `36-confirmconsultationpayment.md`
- [x] Create `37-getmyconsultations.md`
- [x] Create `38-generatevideotoken.md`
- [x] Create `39-endconsultation.md`
- [x] Create `40-createconsultationreview.md`
- [x] Create `41-getconsultationreview.md`
- [x] Create `42-getallconsultations.md`
- [x] Create `43-getconsultationbyid.md`

### Per-file skeleton checklist

For each new file created:

- [x] Add title line in the same style as existing sheets.
- [x] Add source workbook line.
- [x] Add sheet name line.
- [x] Add used range line as placeholder if final range is not known yet.
- [x] Add matrix table skeleton with row/column headers matching workbook style.

### Phase 2 exit check

- [x] All `21` to `43` Markdown files exist before detailed matrix authoring begins.

## Phase 3. Author Batch 1: Expert Discovery and Setup

Status: `Completed`

Goal:
Complete the Consultation sheets related to expert browsing, expert profile, and expert-side setup/history.

### Batch 1 target files

- [x] `21-getexperts.md`
- [x] `22-getexpertprofile.md`
- [x] `23-getexpertreviews.md`
- [x] `24-getexperttimeslots.md`
- [x] `25-updateexpertsettings.md`
- [x] `26-createbulktimeslots.md`
- [x] `27-getexpertconsultations.md`

### Per-sheet authoring checklist

For each Batch 1 file:

- [x] Fill header metadata.
- [x] Fill lines-of-code placeholder strategy or note.
- [x] Write a concrete test requirement summary.
- [x] Define precondition rows.
- [x] Define input condition rows.
- [x] Define confirmation rows.
- [x] Add UTCID columns.
- [x] Mark each test case type as `N`, `A`, or `B`.
- [x] Leave execution result rows in a state consistent with existing report convention.

### Batch 1 verification checklist

- [x] Public endpoints distinguish optional auth vs required auth correctly.
- [x] Expert-only endpoints distinguish `401` vs `403` cases where applicable.
- [x] Validation cases are based on verified request rules.
- [x] Pagination/filter cases are included where the endpoint supports them.

### Phase 3 exit check

- [x] All Batch 1 files have complete matrix content, not placeholders.

## Phase 4. Author Batch 2: Scheduled Consultation Flow

Status: `Completed`

Goal:
Complete scheduled booking creation, scheduled list retrieval, and scheduled payment coverage.

### Batch 2 target files

- [x] `28-createconsultationbooking.md`
- [x] `29-getmyscheduledbookings.md`
- [x] `30-getexpertscheduledbookings.md`
- [x] `31-payscheduledbooking.md`
- [x] `36-confirmconsultationpayment.md`

### Batch 2 content checklist

- [x] Capture slot existence cases.
- [x] Capture slot availability cases.
- [x] Capture started/past-slot cases.
- [x] Capture auth and role cases.
- [x] Capture wallet payment happy path.
- [x] Capture PayOS pending path.
- [x] Capture payment confirmation path.
- [x] Capture invalid ID / not found cases.
- [x] Capture list filtering behavior where verified in code/tests.

### Batch 2 verification checklist

- [x] Scheduled booking matrix aligns with `BookingStatus.PendingPayment` initial state.
- [x] Payment matrix reflects wallet vs PayOS differences.
- [x] Confirm-payment matrix does not duplicate scheduled payment matrix without purpose.
- [x] Expert scheduled list matrix reflects code-verified filter behavior.

### Phase 4 exit check

- [x] All Batch 2 files are complete and aligned with distilled docs and tests.

## Phase 5. Author Batch 3: Emergency Consultation Flow

Status: `Completed`

Goal:
Complete the emergency request lifecycle from creation to payment to expert response.

### Batch 3 target files

- [x] `32-createemergencyconsultationrequest.md`
- [x] `33-payemergencyrequest.md`
- [x] `34-acceptemergencyconsultationrequest.md`
- [x] `35-rejectemergencyconsultationrequest.md`

### Batch 3 content checklist

- [x] Capture request creation success path.
- [x] Capture expert existence or invalid target cases if verified.
- [x] Capture payment success path.
- [x] Capture offline expert blocking behavior.
- [x] Capture request status transition to `PendingExpertResponse`.
- [x] Capture accept success path.
- [x] Capture reject success path.
- [x] Capture wrong expert / forbidden cases.
- [x] Capture invalid request ID / not found cases.
- [x] Capture invalid status transition cases.

### Batch 3 verification checklist

- [x] Acceptance matrix reflects consultation creation and ongoing status.
- [x] Rejection matrix reflects refund behavior where appropriate.
- [x] Emergency TTL-dependent behaviors are only documented when code-verified.

### Phase 5 exit check

- [x] All Batch 3 files are complete and internally consistent.

## Phase 6. Author Batch 4: Active Consultation and Review Flow

Status: `Pending`

Goal:
Complete the matrices for in-progress consultation usage and post-consultation review.

### Batch 4 target files

- [ ] `37-getmyconsultations.md`
- [ ] `38-generatevideotoken.md`
- [ ] `39-endconsultation.md`
- [ ] `40-createconsultationreview.md`
- [ ] `41-getconsultationreview.md`

### Batch 4 content checklist

- [ ] Capture consultation history success path.
- [ ] Capture filter and pagination cases.
- [ ] Capture video token success path.
- [ ] Capture participant/admin authorization rules for video token.
- [ ] Capture room-not-initialized validation case.
- [ ] Capture end consultation success path.
- [ ] Capture review create validation rules.
- [ ] Capture review retrieval with existing review.
- [ ] Capture review retrieval with `data = null`.

### Batch 4 verification checklist

- [ ] Review matrix separates missing review from hard error.
- [ ] Video token matrix distinguishes `404`, `403`, and validation cases.
- [ ] End consultation matrix reflects current service-driven authorization behavior.

### Phase 6 exit check

- [ ] All Batch 4 files are complete and evidence-backed.

## Phase 7. Author Batch 5: Admin Monitoring

Status: `Pending`

Goal:
Complete admin-facing Consultation monitoring matrices.

### Batch 5 target files

- [ ] `42-getallconsultations.md`
- [ ] `43-getconsultationbyid.md`

### Batch 5 content checklist

- [ ] Capture admin authorization requirement.
- [ ] Capture list success path.
- [ ] Capture filter cases for status and type.
- [ ] Capture pagination behavior.
- [ ] Capture detail success path.
- [ ] Capture not found case.
- [ ] Capture scheduled vs emergency shape differences where client-visible.
- [ ] Capture nullable side-record behavior only if it is verified and important.

### Batch 5 verification checklist

- [ ] Admin list matrix reflects merged scheduled/emergency behavior only when it affects output.
- [ ] Admin detail matrix separates scheduled-only fields from emergency-only fields.

### Phase 7 exit check

- [ ] Both admin files are complete and consistent with admin distilled docs.

## Phase 8. Cross-Sheet Consistency Pass

Status: `Pending`

Goal:
Detect and fix structural drift before workbook write-back.

### Phase 8 checklist

- [ ] Check all Consultation files use the same sheet metadata format.
- [ ] Check all titles use the intended function names.
- [ ] Check all source workbook links are correct.
- [ ] Check all sheet names are unique.
- [ ] Check numbering from `21` to `43` is continuous.
- [ ] Check `03-functions.md` sheet names match actual filenames.
- [ ] Check no sheet still contains placeholder text copied from older flows.
- [ ] Check UTCID numbering inside each sheet is continuous and readable.
- [ ] Check `N/A/B` count rows are internally coherent.
- [ ] Check result rows are not accidentally pre-filled with final execution claims unless intentionally preserved by convention.

### Phase 8 exit check

- [ ] Markdown set is coherent enough to transfer into workbook without interpretation work.

## Phase 9. Workbook Write-Back

Status: `Pending`

Goal:
Transfer the completed Consultation Markdown set into the Excel workbook.

### Phase 9 checklist

- [ ] Confirm all previous phases are fully checked.
- [ ] Add Consultation function rows into workbook `Functions` sheet.
- [ ] Insert Consultation sheets in the workbook in the agreed order.
- [ ] Copy matrix content from each Markdown file into the matching workbook sheet.
- [ ] Update any formulas, links, or summary references affected by new sheets.
- [ ] Recheck workbook sheet order.
- [ ] Recheck workbook navigation links if they exist.
- [ ] Recheck summary/statistics formulas if they depend on function count or sheet range.

### Phase 9 exit check

- [ ] Workbook contains the full Consultation section.
- [ ] Workbook structure matches Markdown mirror.

## Evidence Mapping Guidance

Use these evidence sources while authoring matrices:

- Distilled docs for business rules and API contract.
- Controller code for route/auth/method grounding.
- Integration tests for realistic success and failure scenarios.
- Property tests for invariant-style conditions.
- Existing workbook Markdown sheets for matrix structure only.

## Decision Log

- [x] Use Markdown-first execution and postpone workbook edits until the Consultation set is complete.
- [x] Include core Consultation business APIs in the first pass.
- [x] Exclude helper endpoints and realtime hubs as standalone report sheets in the first pass.
- [x] Start Consultation sheet numbering after current last sheet `20`.
- [x] No deviation recorded at Phase 1. Consultation Markdown filenames remain mapped to sheet numbers `21` to `43`.

## Blockers / Notes

- [x] `05-statistics.md` currently contains existing `#REF!` values and will require formula review during Phase 9 workbook write-back.

If a blocker appears, add it here in this format:

- [ ] Blocker: `<short description>`
- Impact: `<what cannot continue>`
- Next action: `<what is needed to unblock>`
