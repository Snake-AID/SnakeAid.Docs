# AGENT MEMORY: Usage Guide Generation Protocol
# SnakeAid — Usage Guide Specialist

> SYSTEM INSTRUCTION (STRICT)
> This file defines the protocol for generating `*.usageguide.md` files ONLY.
> You MUST follow this protocol when creating or updating usage guide documentation.
> This is a specialized subset of the main AGENTS.md protocol.

---

# Purpose
This agent specializes in generating external-facing API documentation for consumers.
It produces ONLY `<module>.usageguide.md` files.

---

# When to Use This Agent
Use this agent when:
- A new module has been implemented and needs consumer documentation
- An existing API contract has changed (endpoints, request/response formats, status codes)
- Error catalogs need to be updated
- Authentication requirements have changed
- You need to document public API surface for external consumers

Do NOT use this agent for:
- Internal architecture documentation (use main AGENTS.md)
- Source code documentation (use main AGENTS.md)
- Operation planning (use main AGENTS.md)
- Introduction/domain context (use main AGENTS.md)

---

# Input Requirements
Before generating a usage guide, you MUST have:

1. **Module Location**: Full path to the module (e.g., `02-layers/aspnet-identity/`)
2. **Source Code Reference**: Access to `<module>.sourcecode.md` (the source of truth)
3. **Implementation Status**: Confirmation that the code is implemented and stable

---

# Usage Guide Structure

## Required Sections

### 1. Overview
Brief description of what the module provides to consumers.
- Purpose from consumer perspective
- Key capabilities
- Prerequisites (auth, permissions, etc.)

### 2. Authentication & Authorization
- Required authentication method (JWT, API key, etc.)
- Required roles/permissions
- Token format and headers
- Example auth headers

### 3. Endpoints
For each endpoint:
```markdown
#### `[METHOD] /api/path/to/endpoint`

**Description**: What this endpoint does

**Authentication**: Required | Optional | None

**Required Permissions**: List of roles/permissions

**Request Headers**:
| Header | Type | Required | Description |
|--------|------|----------|-------------|
| Authorization | string | Yes | Bearer {token} |

**Path Parameters**:
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | int | Yes | Resource identifier |

**Query Parameters**:
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| page | int | No | 1 | Page number |

**Request Body**:
```json
{
  "field": "value",
  "nested": {
    "field": "value"
  }
}
```

**Request Body Schema**:
| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| field | string | Yes | max 100 chars | Field description |

**Success Response** (200 OK):
```json
{
  "data": {
    "id": 1,
    "field": "value"
  },
  "meta": {
    "timestamp": "2024-01-01T00:00:00Z"
  }
}
```

**Error Responses**:
See Error Catalog section below.
```

### 4. Data Models
Document all DTOs/models used in requests/responses:
```markdown
### ModelName
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| id | int | Yes | Unique identifier |
| name | string | Yes | Display name |
| createdAt | datetime | Yes | ISO 8601 format |
```

### 5. Error Catalog
Comprehensive list of all possible errors:
```markdown
| Status Code | Error Code | Message | Description | Resolution |
|-------------|------------|---------|-------------|------------|
| 400 | INVALID_INPUT | Invalid request format | Request body validation failed | Check request schema |
| 401 | UNAUTHORIZED | Authentication required | Missing or invalid token | Provide valid Bearer token |
| 403 | FORBIDDEN | Insufficient permissions | User lacks required role | Contact admin for access |
| 404 | NOT_FOUND | Resource not found | Requested resource doesn't exist | Verify resource ID |
| 409 | CONFLICT | Resource conflict | Duplicate or state conflict | Check resource state |
| 500 | INTERNAL_ERROR | Internal server error | Unexpected server error | Contact support |
```

### 6. Rate Limiting (if applicable)
- Rate limit rules
- Headers returned
- How to handle rate limit errors

### 7. Webhooks (if applicable)
- Webhook endpoints
- Event types
- Payload formats
- Retry logic

### 8. Examples
Real-world usage examples:
```markdown
### Example: Create a Resource

**Request**:
```bash
curl -X POST https://api.example.com/api/resources \
  -H "Authorization: Bearer {{TOKEN}}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Example Resource",
    "type": "standard"
  }'
```

**Response**:
```json
{
  "data": {
    "id": 123,
    "name": "Example Resource",
    "type": "standard",
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```
```

### 9. SDK/Client Libraries (if applicable)
- Available SDKs
- Installation instructions
- Basic usage examples

### 10. Changelog (optional)
- Recent API changes
- Deprecation notices
- Migration guides

---

# Required Frontmatter

```yaml
---
doc_role: baseline
module: <module-name>
kind: <layer|flow>
doc_type: usageguide
status: active
last_updated: YYYY-MM-DD
api_version: v1
owners: [backend-team]
---
```

---

# Generation Rules

## DO:
- ✅ Extract ALL public endpoints from `<module>.sourcecode.md`
- ✅ Document ONLY implemented features (no TODOs, no future plans)
- ✅ Provide complete request/response examples
- ✅ Include ALL possible error codes
- ✅ Use realistic example data (not "string", "number", etc.)
- ✅ Specify exact data types (int32, int64, string, datetime, etc.)
- ✅ Document all constraints (max length, regex patterns, enums)
- ✅ Use placeholders for secrets: `{{TOKEN}}`, `{{API_KEY}}`
- ✅ Include HTTP status codes for all responses
- ✅ Document pagination format if applicable
- ✅ Specify datetime formats (ISO 8601, Unix timestamp, etc.)
- ✅ Include Content-Type headers in examples

## DO NOT:
- ❌ Include internal implementation details
- ❌ Reference internal class names or namespaces
- ❌ Document private/internal endpoints
- ❌ Include database schema details
- ❌ Reference internal services or dependencies
- ❌ Include TODO items or future plans
- ❌ Use vague descriptions ("returns data", "processes request")
- ❌ Omit error cases
- ❌ Include actual secrets or credentials
- ❌ Document deprecated endpoints without migration path

---

# Quality Checklist

Before finalizing a usage guide, verify:

- [ ] All public endpoints are documented
- [ ] Every endpoint has at least one complete example
- [ ] All request parameters are documented with types and constraints
- [ ] All response fields are documented
- [ ] Error catalog includes all possible error codes
- [ ] Authentication requirements are clear
- [ ] No internal implementation details leaked
- [ ] No secrets or credentials included
- [ ] Examples use realistic data
- [ ] HTTP methods and status codes are correct
- [ ] Datetime formats are specified
- [ ] Pagination is documented (if applicable)
- [ ] Rate limiting is documented (if applicable)
- [ ] Frontmatter is complete and correct

---

# Update Triggers

Update the usage guide when:
- New endpoints are added
- Endpoint signatures change (parameters, request/response format)
- New error codes are introduced
- Authentication/authorization rules change
- Rate limiting rules change
- API version changes
- Deprecations occur

---

# Example Workflow

1. **Receive Request**: "Generate usage guide for aspnet-identity module"

2. **Locate Module**: `02-layers/aspnet-identity/`

3. **Read Source**: Load `aspnet-identity.sourcecode.md`

4. **Extract Public API**:
   - Identify all public endpoints
   - Extract request/response schemas
   - Identify error codes
   - Note authentication requirements

5. **Generate Usage Guide**: Create `aspnet-identity.usageguide.md` following the structure above

6. **Validate**: Run through quality checklist

7. **Save**: Write file with proper frontmatter

---

# Security Rules

Never include in usage guides:
- ❌ Actual JWT secrets or signing keys
- ❌ Database connection strings
- ❌ Internal service URLs (use `https://api.example.com`)
- ❌ Real user credentials
- ❌ API keys or tokens
- ❌ Internal IP addresses
- ❌ Infrastructure details

Always use placeholders:
- `{{TOKEN}}` for bearer tokens
- `{{API_KEY}}` for API keys
- `{{USER_ID}}` for user identifiers
- `https://api.example.com` for base URLs

---

# Encoding Rules
- All markdown files MUST be UTF-8 (no BOM)
- If mojibake occurs, repair once with `ftfy.fix_encoding`
- Do not keep alternate encodings in the repo

---

# Context Loading Protocol

When generating a usage guide:

1. Locate module under `01-flows/` or `02-layers/`
2. Read `<module>.sourcecode.md` (REQUIRED)
3. Optionally read `<module>.introduction.md` for context
4. Extract public API surface
5. Generate `<module>.usageguide.md`
6. Validate against quality checklist
7. Save with proper frontmatter

---

# Mental Model

```
sourcecode.md (internal truth)
    ↓
  [Extract Public API]
    ↓
usageguide.md (external contract)
```

The usage guide is a **filtered view** of the source code, showing only what external consumers need to know.

---

# Relationship to Main AGENTS.md

This agent is a **specialized subset** of the main AGENTS.md protocol.

- Main AGENTS.md: Handles full documentation lifecycle (introduction, sourcecode, usageguide, operations)
- AGENTS-USAGEGUIDE.md: Handles ONLY usageguide generation

When in doubt, refer to main AGENTS.md for:
- Module structure
- Operation lifecycle
- Baseline vs operations distinction
- Frontmatter standards

---

# Success Criteria

A well-generated usage guide enables a developer to:
1. Understand what the API does (without reading code)
2. Authenticate successfully
3. Make valid requests
4. Handle all error cases
5. Integrate the API into their application

If a developer needs to ask "how do I...?", the usage guide is incomplete.

---

END OF PROTOCOL
