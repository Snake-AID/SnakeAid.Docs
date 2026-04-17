# Report 4 Database Design Table Descriptions Plan

## Objective

Complete the `Table Descriptions` subsection under `## 2. Database Design` in [SnakeAid Report4 DatabaseDesign.md](D:/SourceCode/Snake_AID/SnakeAid.Docs/Docs/11-SoftwareDocument/Report%204/Database%20Design/SnakeAid%20Report4%20DatabaseDesign.md) with concise, table-by-table descriptions that are grounded in the current physical ERD and cross-checked against available backend documentation.

## Current State

- The target document currently contains only a placeholder table for `Table Descriptions`.
- The main structural source is the Report 4 ERD in:
  - `D:\SourceCode\Snake_AID\SnakeAid.Docs\Docs\11-SoftwareDocument\Report 4\Database Design\SnakeAid Physical ERD.mermaid.md`
- The ERD already enumerates the physical tables, key fields, and many FK relationships.
- The backend docs folder is available at:
  - `D:\SourceCode\Snake_AID\SnakeAid.Docs\Docs\05-Backend`
- Reliability of backend docs is uneven:
  - Consultation flow documents are considered up to date.
  - Other flows and layer docs may be partially outdated and must be treated as supporting evidence only.

## Working Direction

The table descriptions should be produced with this evidence priority:

1. Report 4 physical ERD
2. Current backend code and schema behavior when needed
3. Backend docs in `Docs/05-Backend` as supporting context

The writing target is not to reproduce every column. It is to provide a reader-friendly description of what each table stores, plus the most important key structure:

- business purpose of the table
- primary key
- foreign keys that define its main relationships
- role of the table in the broader feature flow when that role is clear

## Scope Baseline

The ERD currently includes a broad platform-wide database surface, including at least these domains:

- identity and profile data
- consultation and chat
- snakebite incidents and rescue dispatch
- snake catching requests and missions
- snake library and venom knowledge
- AI recognition and media storage
- payments, wallets, and withdrawals
- notifications, settings, and operational support tables

This means the final `Table Descriptions` section will likely be large. The work should therefore be executed in batches instead of trying to write every table in one uninterrupted pass.

## Practical Constraints

- Use the ERD as the canonical starting point for table names and relationships.
- Do not trust older docs blindly when they conflict with the ERD or current code.
- Keep descriptions compact and report-friendly.
- Avoid speculative statements about behavior that cannot be supported by the ERD, code, or still-reliable docs.
- Preserve report formatting so the final content can be pasted into the existing Report 4 section with minimal restructuring.

## Expected Output Shape

Each row in `Table Descriptions` should ultimately follow this pattern:

- `No`
- `Table`
- `Description`

And each description should generally include:

- what the table stores
- primary key field(s)
- foreign key field(s), if any

## Recommended Authoring Strategy

1. Inventory all table names from the ERD.
2. Group them by domain to reduce context switching.
3. Draft short descriptions directly from the ERD first.
4. Use backend docs only to clarify business meaning.
5. Use backend code when a table meaning is unclear or the docs look stale.
6. Merge the verified descriptions into the Report 4 Markdown file only after the batch is stable.

## Definition of Done

This work is complete only when:

- every ERD table has a non-placeholder description
- key PK/FK information is explicitly mentioned
- wording is consistent across rows
- obviously outdated or contradictory doc claims have been filtered out
- the `Table Descriptions` block in Report 4 is ready for direct review
