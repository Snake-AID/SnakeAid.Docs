---
doc_role: baseline
module: nosql-report-media
kind: layer
status: active
last_updated: 2026-02-23
owners: [backend-team]
---

# NoSql Report Media (ReportMedia)

## Domain Context

The SnakeAid backend relies on a centralized system for storing images, videos, and audio (Media) associated with various entities across the platform. Instead of creating a separate Media table for each domain entity (e.g., `SnakeCatchingRequestMedia`, `SnakebiteIncidentMedia`, `CommunityReportMedia`), the project utilizes a single, unified `ReportMedia` table.

## Business Rules & Design Pattern

This layer implements a **NoSql Association Pattern** (also known as a DocumentDB-style reference pattern within a relational SQL DB).

- **Unified Table**: All media uploads are stored in the `ReportMedias` table.
- **Dynamic Linking**: The `ReportMedia` table uses two columns, `ReferenceId` (Guid) and `ReferenceType` (Enum: `MediaReferenceType`), to establish relationships with its parent entities.
- **Decoupled Relational Constraints**: Explicit Database Foreign Keys CANNOT be used here. A single `ReferenceId` column cannot logically point to multiple different tables simultaneously. Therefore, the DB layer treats `ReferenceId` as a regular scalar Guid.

## Scope

- Covers the entity model `ReportMedia`.
- Covers the core API `UploadReportMediaAsync` which accepts a generic payload.
- In-Scope: How parent entities query and manage their media.
- Out-of-Scope: Third-party storage integration (e.g., Cloudinary), which belongs to the `cloudinary` layer.
