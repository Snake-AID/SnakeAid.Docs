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
- [ ] Created Consultation Markdown sheet files.
- [ ] Populated Consultation matrix content.
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

Status: `Pending`

Goal:
Create the full Markdown skeleton for every Consultation sheet so content work can resume file-by-file without ambiguity.

### Phase 2 checklist

- [ ] Create `21-getexperts.md`
- [ ] Create `22-getexpertprofile.md`
- [ ] Create `23-getexpertreviews.md`
- [ ] Create `24-getexperttimeslots.md`
- [ ] Create `25-updateexpertsettings.md`
- [ ] Create `26-createbulktimeslots.md`
- [ ] Create `27-getexpertconsultations.md`
- [ ] Create `28-createconsultationbooking.md`
- [ ] Create `29-getmyscheduledbookings.md`
- [ ] Create `30-getexpertscheduledbookings.md`
- [ ] Create `31-payscheduledbooking.md`
- [ ] Create `32-createemergencyconsultationrequest.md`
- [ ] Create `33-payemergencyrequest.md`
- [ ] Create `34-acceptemergencyconsultationrequest.md`
- [ ] Create `35-rejectemergencyconsultationrequest.md`
- [ ] Create `36-confirmconsultationpayment.md`
- [ ] Create `37-getmyconsultations.md`
- [ ] Create `38-generatevideotoken.md`
- [ ] Create `39-endconsultation.md`
- [ ] Create `40-createconsultationreview.md`
- [ ] Create `41-getconsultationreview.md`
- [ ] Create `42-getallconsultations.md`
- [ ] Create `43-getconsultationbyid.md`

### Per-file skeleton checklist

For each new file created:

- [ ] Add title line in the same style as existing sheets.
- [ ] Add source workbook line.
- [ ] Add sheet name line.
- [ ] Add used range line as placeholder if final range is not known yet.
- [ ] Add matrix table skeleton with row/column headers matching workbook style.

### Phase 2 exit check

- [ ] All `21` to `43` Markdown files exist before detailed matrix authoring begins.

## Phase 3. Author Batch 1: Expert Discovery and Setup

Status: `Pending`

Goal:
Complete the Consultation sheets related to expert browsing, expert profile, and expert-side setup/history.

### Batch 1 target files

- [ ] `21-getexperts.md`
- [ ] `22-getexpertprofile.md`
- [ ] `23-getexpertreviews.md`
- [ ] `24-getexperttimeslots.md`
- [ ] `25-updateexpertsettings.md`
- [ ] `26-createbulktimeslots.md`
- [ ] `27-getexpertconsultations.md`

### Per-sheet authoring checklist

For each Batch 1 file:

- [ ] Fill header metadata.
- [ ] Fill lines-of-code placeholder strategy or note.
- [ ] Write a concrete test requirement summary.
- [ ] Define precondition rows.
- [ ] Define input condition rows.
- [ ] Define confirmation rows.
- [ ] Add UTCID columns.
- [ ] Mark each test case type as `N`, `A`, or `B`.
- [ ] Leave execution result rows in a state consistent with existing report convention.

### Batch 1 verification checklist

- [ ] Public endpoints distinguish optional auth vs required auth correctly.
- [ ] Expert-only endpoints distinguish `401` vs `403` cases where applicable.
- [ ] Validation cases are based on verified request rules.
- [ ] Pagination/filter cases are included where the endpoint supports them.

### Phase 3 exit check

- [ ] All Batch 1 files have complete matrix content, not placeholders.

## Phase 4. Author Batch 2: Scheduled Consultation Flow

Status: `Pending`

Goal:
Complete scheduled booking creation, scheduled list retrieval, and scheduled payment coverage.

### Batch 2 target files

- [ ] `28-createconsultationbooking.md`
- [ ] `29-getmyscheduledbookings.md`
- [ ] `30-getexpertscheduledbookings.md`
- [ ] `31-payscheduledbooking.md`
- [ ] `36-confirmconsultationpayment.md`

### Batch 2 content checklist

- [ ] Capture slot existence cases.
- [ ] Capture slot availability cases.
- [ ] Capture started/past-slot cases.
- [ ] Capture auth and role cases.
- [ ] Capture wallet payment happy path.
- [ ] Capture PayOS pending path.
- [ ] Capture payment confirmation path.
- [ ] Capture invalid ID / not found cases.
- [ ] Capture list filtering behavior where verified in code/tests.

### Batch 2 verification checklist

- [ ] Scheduled booking matrix aligns with `BookingStatus.PendingPayment` initial state.
- [ ] Payment matrix reflects wallet vs PayOS differences.
- [ ] Confirm-payment matrix does not duplicate scheduled payment matrix without purpose.
- [ ] Expert scheduled list matrix reflects code-verified filter behavior.

### Phase 4 exit check

- [ ] All Batch 2 files are complete and aligned with distilled docs and tests.

## Phase 5. Author Batch 3: Emergency Consultation Flow

Status: `Pending`

Goal:
Complete the emergency request lifecycle from creation to payment to expert response.

### Batch 3 target files

- [ ] `32-createemergencyconsultationrequest.md`
- [ ] `33-payemergencyrequest.md`
- [ ] `34-acceptemergencyconsultationrequest.md`
- [ ] `35-rejectemergencyconsultationrequest.md`

### Batch 3 content checklist

- [ ] Capture request creation success path.
- [ ] Capture expert existence or invalid target cases if verified.
- [ ] Capture payment success path.
- [ ] Capture offline expert blocking behavior.
- [ ] Capture request status transition to `PendingExpertResponse`.
- [ ] Capture accept success path.
- [ ] Capture reject success path.
- [ ] Capture wrong expert / forbidden cases.
- [ ] Capture invalid request ID / not found cases.
- [ ] Capture invalid status transition cases.

### Batch 3 verification checklist

- [ ] Acceptance matrix reflects consultation creation and ongoing status.
- [ ] Rejection matrix reflects refund behavior where appropriate.
- [ ] Emergency TTL-dependent behaviors are only documented when code-verified.

### Phase 5 exit check

- [ ] All Batch 3 files are complete and internally consistent.

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
