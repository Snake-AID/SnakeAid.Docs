# Report 4 Database Design Table Descriptions Roadmap

## Purpose

Track the work required to complete the `Table Descriptions` subsection in Report 4 so the task can be resumed at any time without re-discovering the scope, evidence sources, or progress state.

## Working Rule

- This file is the execution control document for the Report 4 database table description task.
- Progress must be tracked here with checkbox updates.
- The ERD is the first source of truth for table names, PKs, and FKs.
- `Docs/05-Backend` is supporting evidence, not automatic truth.
- Consultation docs are currently the most reliable subset in `Docs/05-Backend`.
- If a table meaning is still unclear after checking the ERD and docs, verify against current backend code before finalizing the description.

## Scope Baseline

- Target report section:
  - `D:\SourceCode\Snake_AID\SnakeAid.Docs\Docs\11-SoftwareDocument\Report 4\Database Design\SnakeAid Report4 DatabaseDesign.md`
- ERD source:
  - `D:\SourceCode\Snake_AID\SnakeAid.Docs\Docs\11-SoftwareDocument\Report 4\Database Design\SnakeAid Physical ERD.mermaid.md`
- Planning workspace:
  - `D:\SourceCode\Snake_AID\SnakeAid.Docs\Docs\11-SoftwareDocument\Report 4\Controller`
- Supporting docs:
  - `D:\SourceCode\Snake_AID\SnakeAid.Docs\Docs\05-Backend`

## Resume Protocol

When resuming:

1. Read `Controller/introduction.md`.
2. Read this roadmap from top to bottom.
3. Find the first phase that is not fully checked.
4. Continue from the first unchecked task in that phase.
5. Update this file before stopping work again.

## Current Status

- [x] Identified the target report file and ERD source.
- [x] Confirmed the `Table Descriptions` section is still a placeholder.
- [x] Confirmed that backend docs are only partially reliable.
- [x] Created planning documents in `Report 4/Controller`.
- [x] Extracted the full table inventory from the ERD.
- [x] Grouped tables into documentation batches.
- [x] Drafted all table descriptions.
- [x] Verified ambiguous or potentially outdated areas.
- [x] Inserted the completed descriptions into the Report 4 Markdown file.
- [x] Performed final consistency review.

## Phase 1. Freeze Sources and Output Format

Status: `Completed`

Goal:
Lock the source priority and the expected output shape before any row drafting starts.

### Phase 1 checklist

- [x] Confirm the target subsection is `## 2. Database Design`.
- [x] Confirm the missing deliverable is the `Table Descriptions` table.
- [x] Confirm the Report 4 ERD is the primary structural source.
- [x] Confirm `Docs/05-Backend` may be outdated outside consultation flow.
- [x] Define the description pattern as:
  - [x] business purpose
  - [x] primary key(s)
  - [x] foreign key(s)

### Phase 1 exit check

- [x] The source-of-truth order is clear before description authoring begins.

## Phase 2. Build ERD Table Inventory

Status: `Completed`

Goal:
Create a complete list of all tables present in the ERD so no table is missed later.

### Phase 2 checklist

- [x] Extract every table name from the ERD.
- [x] Record PK fields for each table.
- [x] Record FK fields for each table where explicit in the ERD.
- [x] Flag tables with ambiguous purpose names for deeper review.
- [x] Count the total number of tables to support progress tracking.

### Phase 2 exit check

- [x] A complete table inventory exists and can be used as the master checklist.

## Phase 3. Group Tables Into Writing Batches

Status: `Completed`

Goal:
Reduce context switching by grouping related tables before drafting descriptions.

### Suggested batches

- [x] Identity, profiles, notifications, and system tables
- [x] Consultation, bookings, chat, and expert domain tables
- [x] Snakebite incident and rescue dispatch tables
- [x] Snake catching request and mission tables
- [x] Snake knowledge, antivenom, venom, and region mapping tables
- [x] AI, media, transaction, wallet, and operational tables

### Phase 3 checklist

- [x] Assign every table to exactly one batch.
- [x] Mark any cross-domain tables that need extra care.
- [x] Decide batch order for authoring.

### Phase 3 exit check

- [x] Every ERD table is assigned to a drafting batch.

## Phase 4. Draft Descriptions Batch by Batch

Status: `Completed`

Goal:
Write report-ready descriptions for every table using the agreed evidence order.

### Per-table drafting checklist

- [x] State what the table stores in one concise sentence.
- [x] State the primary key field(s).
- [x] State the foreign key field(s), if any.
- [x] Keep wording concise and suitable for report format.
- [x] Avoid unsupported business claims.
- [x] Mark tables that still need code verification.

### Batch completion rule

- [x] A batch is complete only when every table in that batch has a usable description.

### Phase 4 exit check

- [x] All ERD tables have draft descriptions, even if some still need verification.

## Phase 5. Verify Ambiguous or Potentially Outdated Areas

Status: `Completed`

Goal:
Resolve tables whose meaning cannot be safely inferred from the ERD alone.

### Verification triggers

- [x] Table name is too generic to infer purpose safely.
- [x] Supporting docs disagree with the ERD.
- [x] Docs appear stale for the relevant feature area.
- [x] FK patterns suggest a meaning that is not obvious from the name alone.

### Phase 5 checklist

- [x] Use backend docs to clarify business context where still reliable.
- [x] Use current backend code for unresolved tables.
- [x] Normalize wording after verification so style stays consistent.

### Phase 5 exit check

- [x] No table description depends on an unverified guess.

## Phase 6. Insert Into Report 4 Markdown

Status: `Completed`

Goal:
Replace the placeholder content in the Report 4 file with the completed `Table Descriptions` table.

### Phase 6 checklist

- [x] Preserve the existing report structure.
- [x] Replace the example placeholder rows.
- [x] Insert all finalized table rows.
- [x] Keep numbering continuous.
- [x] Keep table formatting readable in Markdown.

### Phase 6 exit check

- [x] The Report 4 Markdown file contains a complete non-placeholder `Table Descriptions` section.

## Phase 7. Final Review

Status: `Completed`

Goal:
Check quality, consistency, and completeness before considering the task done.

### Phase 7 checklist

- [x] Confirm every ERD table is represented exactly once.
- [x] Confirm PK/FK information is present in every applicable row.
- [x] Confirm wording style is consistent across rows.
- [x] Confirm no row still contains placeholder text.
- [x] Confirm no description relies on obviously outdated documentation.
- [x] Confirm the final section is ready for direct academic/report review.

### Phase 7 exit check

- [x] The `Table Descriptions` section can be reviewed without additional reconstruction work.

## Decision Log

- [x] Do not fill the `Table Descriptions` section immediately; create planning documents first.
- [x] Keep all planning documents in English.
- [x] Use the ERD as the primary database-design source.
- [x] Treat non-consultation backend docs as potentially outdated supporting material.
- [x] Record the ERD inventory in `Controller/table-inventory.md` before starting table description writing.
- [x] Record batch assignments in `Controller/table-batches.md` before drafting final report wording.

## Blockers / Notes

Add blockers here if they appear during execution:

- [ ] Blocker: `<short description>`
- Impact: `<what cannot continue>`
- Next action: `<what is needed to unblock>`

## Final Review Notes

- `TABLE_ROWS=51` in the final Report 4 table description block.
- No missing tables compared with `Controller/table-inventory.md`.
- No extra tables compared with `Controller/table-inventory.md`.
- No placeholder sample text remains in the `Database Design` section.
