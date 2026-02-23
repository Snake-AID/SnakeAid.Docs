---
doc_role: baseline
module: nosql-report-media
kind: layer
status: active
last_updated: 2026-02-23
owners: [backend-team]
---

# NoSql Report Media - Source Code Reference

## Entity Schema (`ReportMedia.cs`)

The `ReportMedia` entity acts as a nosql child to multiple tables.

```csharp
public class ReportMedia : BaseEntity
{
    [Key]
    public Guid Id { get; set; }

    // NoSql keys (NOT constrained by DB Foreign Key)
    public Guid ReferenceId { get; set; }        // ID of the parent entity
    public MediaReferenceType ReferenceType { get; set; } // Type of the parent entity

    public MediaPurpose Purpose { get; set; }

    // ... URLs and metadata
}
```

## Entity Framework Core Invariants (CRITICAL)

Because `ReferenceId` is nosql, **Entity Framework Core cannot automatically track it as a Foreign Key**.
If you declare an `ICollection<ReportMedia>` in a parent entity to facilitate C# coding:

```csharp
public class SnakebiteIncident : BaseEntity
{
    // ...
    public ICollection<ReportMedia> Media { get; set; } = new List<ReportMedia>();
}
```

**YOU MUST explicitly instruct EF Core to Ignore this property** in the Entity's Configuration fluent API, otherwise EF Core will implicitly create DB shadow columns (like `SnakebiteIncidentId`) in the database schemas to satisfy normal relational behavior.

```csharp
// Inside SnakebiteIncidentConfiguration.cs
builder.Ignore(i => i.Media);
```

## Relationships & Query Patterns

Since EF Core is instructed to ignore the `Media` navigation property at the DB Schema level, **you cannot use `.Include(x => x.Media)`** in your LINQ queries. Doing so will throw a runtime `InvalidOperationException`.

**How to query parent entities and their Media:**
Instead of manually fetching the media for each entity (which leads to N+1 query problems) or writing complex explicit queries, we use an elegant Extension Method approach.

1. **Implement `IHasReportMedia` Interface**
   The target entity must implement `IHasReportMedia`, which guarantees it has an `Id` and a `Media` collection.

```csharp
public interface IHasReportMedia
{
    Guid Id { get; set; }
    ICollection<ReportMedia> Media { get; set; }
}

public class SnakebiteIncident : BaseEntity, IHasReportMedia
{
    // ...
}
```

2. **Use the `AttachReportMediaAsync` Extension Method**
   Fetch your entities as normal (without `.Include(x => x.Media)`). Then call `.AttachReportMediaAsync()` on the result.
   This method performs a single DB query to fetch all media for the entities and attaches them in-memory, solving the N+1 problem efficiently.

```csharp
// 1. Fetch parent entities (e.g., a list or a single entity)
var incidents = await _unitOfWork.GetRepository<SnakebiteIncident>()
    .GetListAsync(); // No .Include(x => x.Media) here!

// 2. Attach nosql media in one optimized batch query
await incidents.AttachReportMediaAsync(_unitOfWork, MediaReferenceType.SnakebiteIncident);

// Now incidents[0].Media is fully populated!
```
