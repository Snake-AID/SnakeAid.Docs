---
doc_role: baseline
module: polymorphic-media
kind: layer
status: active
last_updated: 2026-02-23
owners: [backend-team]
---

# NoSql Report Media Usage Guide

## The `ReferenceId` Contract

When working with `ReportMedia`, the backend system must consistently enforce the mapping between `ReferenceId` and `MediaReferenceType`. Because DB-layer constraints are omitted in nosql designs, the application code bears the validation and integrity responsibilities.

### Required Fields for Uploading Media

When uploading a media via API (`POST /api/media/report`), the generic file upload payload requires the following properties directly or via query parameters to ensure consistency:

- `ReferenceId` - The parsed identity Guid of the target parent row in the corresponding table.
- `MediaReferenceType` - The categorized destination Enum (e.g., `SnakeCatchingRequest = 1`).
- `MediaPurpose` - The intent of the media file (e.g., `SnakeIdentification = 1`).

### Checking `ReferenceId` Integrity

Currently, the upload API (`MediaService.UploadReportMediaAsync`) **relies on client accuracy** and does NOT query parent databases (like `SnakebiteIncidents`, `SnakeCatchingRequests`, or `RescueMissions`) to ensure the `ReferenceId` points to an actual existing entity.

In future iterations, adding an abstraction layer to validate `ReferenceId` existence against its provided `MediaReferenceType` repository is recommended to prevent orphan media files from being instantiated.

### Refactoring Guidance for DB Queries

Never use `Include(r => r.Media)` directly on target child APIs. Use the `AttachReportMediaAsync` extension method to fetch media collections in a batched, N+1-safe way, as outlined in `nosql-report-media.sourcecode.md`.
