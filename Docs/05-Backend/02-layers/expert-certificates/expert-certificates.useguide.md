---
doc_role: planning
module: expert-certificates
kind: flow
doc_type: useguide
status: planning
last_updated: 2026-04-20
api_version: v1
owners: [backend-team]
verification_status: current-state-code-verified-with-planned-api-contract
---

# Expert Certificates Useguide

## 1. Table Of Contents

- [1. Table Of Contents](#1-table-of-contents)
- [2. Overview](#2-overview)
- [3. Authentication & Authorization](#3-authentication--authorization)
- [4. Expert/Member Business + Expert/Member APIs](#4-expertmember-business--expertmember-apis)
- [5. Admin Business + Admin APIs](#5-admin-business--admin-apis)
- [6. Shared Data Models](#6-shared-data-models)
- [7. Verified Endpoint List](#7-verified-endpoint-list)
- [8. Changelog](#8-changelog)

## 2. Overview

Current verified backend behavior:

- `ExpertCertificate` exists in the domain model
- there is currently no active API for expert certificates
- public expert responses already expose `isVerified`
- current backend always returns `isVerified = false` in public expert responses
- expert self profile currently does not expose `isVerified`
- report media currently has no `CertExpert` reference type

Planned module behavior:

- expert can create, list, view, update, and delete own certificates
- admin can create, list, view, update, and delete certificates globally
- admin can set `verificationStatus`
- profile verification becomes a real persisted value through `ExpertProfile.IsVerified`
- public expert profile APIs should reflect actual verification state after implementation

## 3. Authentication & Authorization

### Expert/Member Operations

- JWT Bearer token is required
- only role `Expert` is in scope for self-service certificate APIs
- `Member` is not part of this module
- expert can only access certificates where `ExpertId == current user id`

### Admin Operations

- JWT Bearer token is required
- role `Admin` is required
- admin can access certificates across all experts

## 4. Expert/Member Business + Expert/Member APIs

### 4.1 Current Verified Notes

Current code-verified facts relevant to mobile integration:

- `GET /api/experts/{id}` exists today and includes `isVerified`, but the backend currently hardcodes it to `false`
- `GET /api/experts/me/profile` exists today, but it currently does not expose `isVerified`
- there is no current certificate CRUD endpoint yet

### 4.2 Planned `POST /api/experts/me/certificates`

Purpose:

- create a certificate owned by the current expert

Auth:

- JWT Bearer token is required
- caller must have role `Expert`

Request:

```http
POST /api/experts/me/certificates
Authorization: Bearer <jwt>
Content-Type: application/json
```

```json
{
  "certificateName": "Clinical Toxicology Certificate",
  "issuingOrganization": "Vietnam Medical Association",
  "issueDate": "2024-06-15T00:00:00Z",
  "expiryDate": "2027-06-15T00:00:00Z",
  "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001.pdf"
}
```

Planned backend behavior:

- `expertId` is derived from the JWT, not from the request body
- initial `verificationStatus` is `Pending`
- initial `rejectionReason` is empty
- creating a certificate does not auto-set profile verification to `true`

Success response example:

```json
{
  "status_code": 200,
  "message": "Success",
  "is_success": true,
  "data": {
    "id": "9f5d44f4-0e31-4931-983f-08f0fc8d15f1",
    "expertId": "7aaab7a4-6441-4f6c-88d0-e8451adf6d7b",
    "certificateName": "Clinical Toxicology Certificate",
    "issuingOrganization": "Vietnam Medical Association",
    "issueDate": "2024-06-15T00:00:00Z",
    "expiryDate": "2027-06-15T00:00:00Z",
    "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001.pdf",
    "verificationStatus": "Pending",
    "rejectionReason": "",
    "createdAt": "2026-04-20T08:30:00Z",
    "updatedAt": "2026-04-20T08:30:00Z"
  },
  "error": null
}
```

### 4.3 Planned `GET /api/experts/me/certificates`

Purpose:

- list certificates owned by the current expert

Auth:

- JWT Bearer token is required
- caller must have role `Expert`

Request:

```http
GET /api/experts/me/certificates
Authorization: Bearer <jwt>
```

Success response example:

```json
{
  "status_code": 200,
  "message": "Success",
  "is_success": true,
  "data": [
    {
      "id": "9f5d44f4-0e31-4931-983f-08f0fc8d15f1",
      "expertId": "7aaab7a4-6441-4f6c-88d0-e8451adf6d7b",
      "certificateName": "Clinical Toxicology Certificate",
      "issuingOrganization": "Vietnam Medical Association",
      "issueDate": "2024-06-15T00:00:00Z",
      "expiryDate": "2027-06-15T00:00:00Z",
      "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001.pdf",
      "verificationStatus": "Pending",
      "rejectionReason": ""
    }
  ],
  "error": null
}
```

### 4.4 Planned `GET /api/experts/me/certificates/{certificateId}`

Purpose:

- get detail for one certificate owned by the current expert

Auth:

- JWT Bearer token is required
- caller must have role `Expert`

### 4.5 Planned `PUT /api/experts/me/certificates/{certificateId}`

Purpose:

- update one certificate owned by the current expert

Auth:

- JWT Bearer token is required
- caller must have role `Expert`

Request:

```http
PUT /api/experts/me/certificates/9f5d44f4-0e31-4931-983f-08f0fc8d15f1
Authorization: Bearer <jwt>
Content-Type: application/json
```

```json
{
  "certificateName": "Advanced Clinical Toxicology Certificate",
  "issuingOrganization": "Vietnam Medical Association",
  "issueDate": "2024-06-15T00:00:00Z",
  "expiryDate": "2028-06-15T00:00:00Z",
  "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001-v2.pdf"
}
```

Planned backend note:

- if the project locks the recommended rule, updating a previously verified certificate should reset `verificationStatus` to `Pending`

### 4.6 Planned `DELETE /api/experts/me/certificates/{certificateId}`

Purpose:

- delete one certificate owned by the current expert

Auth:

- JWT Bearer token is required
- caller must have role `Expert`

Success response example:

```json
{
  "status_code": 200,
  "message": "Certificate deleted successfully.",
  "is_success": true,
  "data": true,
  "error": null
}
```

## 5. Admin Business + Admin APIs

### 5.1 Planned `POST /api/admin/expert/certificates`

Purpose:

- create a certificate record for a target expert from the admin side

Auth:

- JWT Bearer token is required
- caller must have role `Admin`

Request:

```http
POST /api/admin/expert/certificates
Authorization: Bearer <jwt>
Content-Type: application/json
```

```json
{
  "expertId": "7aaab7a4-6441-4f6c-88d0-e8451adf6d7b",
  "certificateName": "Clinical Toxicology Certificate",
  "issuingOrganization": "Vietnam Medical Association",
  "issueDate": "2024-06-15T00:00:00Z",
  "expiryDate": "2027-06-15T00:00:00Z",
  "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001.pdf",
  "verificationStatus": "Pending",
  "rejectionReason": ""
}
```

### 5.2 Planned `GET /api/admin/expert/certificates`

Purpose:

- list certificates across all experts

Auth:

- JWT Bearer token is required
- caller must have role `Admin`

Planned query examples:

- `/api/admin/expert/certificates`
- `/api/admin/expert/certificates?expertId=7aaab7a4-6441-4f6c-88d0-e8451adf6d7b`
- `/api/admin/expert/certificates?verificationStatus=Pending`

### 5.3 Planned `GET /api/admin/expert/certificates/{certificateId}`

Purpose:

- get detail for one certificate

Auth:

- JWT Bearer token is required
- caller must have role `Admin`

### 5.4 Planned `PUT /api/admin/expert/certificates/{certificateId}`

Purpose:

- update certificate data and review state

Auth:

- JWT Bearer token is required
- caller must have role `Admin`

Request:

```http
PUT /api/admin/expert/certificates/9f5d44f4-0e31-4931-983f-08f0fc8d15f1
Authorization: Bearer <jwt>
Content-Type: application/json
```

```json
{
  "certificateName": "Advanced Clinical Toxicology Certificate",
  "issuingOrganization": "Vietnam Medical Association",
  "issueDate": "2024-06-15T00:00:00Z",
  "expiryDate": "2028-06-15T00:00:00Z",
  "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001-v2.pdf",
  "verificationStatus": "Verified",
  "rejectionReason": ""
}
```

Planned backend behavior:

- if admin sets `verificationStatus = Verified`, `ExpertProfile.IsVerified` should be recalculated
- if admin sets `verificationStatus = Rejected`, `rejectionReason` should be meaningful for expert-facing review feedback

Success response example:

```json
{
  "status_code": 200,
  "message": "Success",
  "is_success": true,
  "data": {
    "id": "9f5d44f4-0e31-4931-983f-08f0fc8d15f1",
    "expertId": "7aaab7a4-6441-4f6c-88d0-e8451adf6d7b",
    "certificateName": "Advanced Clinical Toxicology Certificate",
    "issuingOrganization": "Vietnam Medical Association",
    "issueDate": "2024-06-15T00:00:00Z",
    "expiryDate": "2028-06-15T00:00:00Z",
    "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001-v2.pdf",
    "verificationStatus": "Verified",
    "rejectionReason": "",
    "createdAt": "2026-04-20T08:30:00Z",
    "updatedAt": "2026-04-20T09:00:00Z"
  },
  "error": null
}
```

### 5.5 Planned `DELETE /api/admin/expert/certificates/{certificateId}`

Purpose:

- delete a certificate globally from the admin side

Auth:

- JWT Bearer token is required
- caller must have role `Admin`

## 6. Shared Data Models

### Planned `CreateExpertCertificateRequest`

| Field | Type | Description |
|------|------|-------------|
| certificateName | string | Required. Certificate display name |
| issuingOrganization | string | Required. Issuer name |
| issueDate | DateTime | Required. Issue date |
| expiryDate | DateTime? | Optional. Expiry date |
| certificateUrl | string | Required. Uploaded file URL |

### Planned `AdminCreateExpertCertificateRequest`

| Field | Type | Description |
|------|------|-------------|
| expertId | Guid | Required. Target expert account id |
| certificateName | string | Required |
| issuingOrganization | string | Required |
| issueDate | DateTime | Required |
| expiryDate | DateTime? | Optional |
| certificateUrl | string | Required |
| verificationStatus | enum | Optional/admin-controlled |
| rejectionReason | string | Optional/admin-controlled |

### Planned `UpdateExpertCertificateRequest`

| Field | Type | Description |
|------|------|-------------|
| certificateName | string | Required |
| issuingOrganization | string | Required |
| issueDate | DateTime | Required |
| expiryDate | DateTime? | Optional |
| certificateUrl | string | Required |

### Planned `AdminUpdateExpertCertificateRequest`

| Field | Type | Description |
|------|------|-------------|
| certificateName | string | Required |
| issuingOrganization | string | Required |
| issueDate | DateTime | Required |
| expiryDate | DateTime? | Optional |
| certificateUrl | string | Required |
| verificationStatus | enum | Required for review updates |
| rejectionReason | string | Optional. Expected when rejecting |

### Planned `ExpertCertificateResponse`

| Field | Type | Description |
|------|------|-------------|
| id | Guid | Certificate id |
| expertId | Guid | Expert account id |
| certificateName | string | Certificate name |
| issuingOrganization | string | Issuer |
| issueDate | DateTime | Issue date |
| expiryDate | DateTime? | Expiry date |
| certificateUrl | string | File URL |
| verificationStatus | enum | `Pending`, `Verified`, or `Rejected` |
| rejectionReason | string | Reason when rejected |
| createdAt | DateTime | Creation timestamp |
| updatedAt | DateTime | Last update timestamp |

## 7. Verified Endpoint List

Current verified related endpoints:

- `GET /api/experts`
- `GET /api/experts/{id}`
- `GET /api/experts/me/profile`
- `PUT /api/experts/me/profile`
- `POST /api/media/upload-file`
- `POST /api/media/report`

Current verified gap:

- there is no active expert certificate endpoint yet

Planned endpoint set:

- `POST /api/experts/me/certificates`
- `GET /api/experts/me/certificates`
- `GET /api/experts/me/certificates/{certificateId}`
- `PUT /api/experts/me/certificates/{certificateId}`
- `DELETE /api/experts/me/certificates/{certificateId}`
- `POST /api/admin/expert/certificates`
- `GET /api/admin/expert/certificates`
- `GET /api/admin/expert/certificates/{certificateId}`
- `PUT /api/admin/expert/certificates/{certificateId}`
- `DELETE /api/admin/expert/certificates/{certificateId}`

## 8. Changelog

### 2026-04-20

- created the planning useguide for expert certificates
- documented the current verified gap where public expert profiles expose `isVerified` but always return `false`
- proposed the initial expert and admin CRUD contract for `ExpertCertificate`
