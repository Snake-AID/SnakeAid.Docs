---
module: expert-certs
last_updated: 2026-04-21
status: planning-contract
audience: mobile-and-frontend
---

# Expert Certificates Use Guide

## 1. Overview

This document describes the planned API contract for expert certificate management and the current backend state verified from the repository.

Current verified state:

- `ExpertCertificate` exists in the domain and persistence model
- there is no active certificate CRUD endpoint yet
- `ExpertProfile.IsVerified` is not yet persisted
- public expert directory currently exposes `isVerified` in response shape but the backend currently hard-codes it to `false`

This guide is therefore split into:

- current active backend facts
- planned API contract for implementation

## 2. Authentication and Authorization

### 2.1 Expert APIs

- actor: authenticated `Expert`
- ownership rule: expert can only access certificates where `ExpertId == current user id`

### 2.2 Admin APIs

- actor: authenticated `Admin`
- scope: admin can access certificates across all experts

## 3. Shared Data Models

## 3.1 Planned `ExpertCertificateResponse`

```json
{
  "id": "9f5d44f4-0e31-4931-983f-08f0fc8d15f1",
  "expertId": "7aaab7a4-6441-4f6c-88d0-e8451adf6d7b",
  "certificateName": "Clinical Toxicology Certificate",
  "issuingOrganization": "Vietnam Poison Control Academy",
  "issueDate": "2025-05-01T00:00:00Z",
  "expiryDate": "2028-05-01T00:00:00Z",
  "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001.pdf",
  "verificationStatus": "Pending",
  "rejectionReason": ""
}
```

Field notes:

- `certificateUrl` is already present in the current domain model
- `verificationStatus` maps to existing enum values `Pending`, `Verified`, `Rejected`
- `rejectionReason` should be empty when status is not `Rejected`

## 3.2 Planned `CreateExpertCertificateRequest`

```json
{
  "certificateName": "Clinical Toxicology Certificate",
  "issuingOrganization": "Vietnam Poison Control Academy",
  "issueDate": "2025-05-01T00:00:00Z",
  "expiryDate": "2028-05-01T00:00:00Z",
  "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001.pdf"
}
```

Field notes:

- `certificateName`: required
- `issuingOrganization`: required
- `issueDate`: required
- `expiryDate`: optional
- `certificateUrl`: required

## 3.3 Planned `AdminCreateExpertCertificateRequest`

```json
{
  "expertId": "7aaab7a4-6441-4f6c-88d0-e8451adf6d7b",
  "certificateName": "Clinical Toxicology Certificate",
  "issuingOrganization": "Vietnam Poison Control Academy",
  "issueDate": "2025-05-01T00:00:00Z",
  "expiryDate": "2028-05-01T00:00:00Z",
  "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001.pdf",
  "verificationStatus": "Pending",
  "rejectionReason": ""
}
```

## 3.4 Planned `UpdateExpertCertificateRequest`

```json
{
  "certificateName": "Advanced Clinical Toxicology Certificate",
  "issuingOrganization": "Vietnam Poison Control Academy",
  "issueDate": "2025-05-01T00:00:00Z",
  "expiryDate": "2028-05-01T00:00:00Z",
  "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001-v2.pdf"
}
```

## 3.5 Planned `AdminUpdateExpertCertificateRequest`

```json
{
  "certificateName": "Advanced Clinical Toxicology Certificate",
  "issuingOrganization": "Vietnam Poison Control Academy",
  "issueDate": "2025-05-01T00:00:00Z",
  "expiryDate": "2028-05-01T00:00:00Z",
  "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001-v2.pdf",
  "verificationStatus": "Verified",
  "rejectionReason": ""
}
```

## 3.6 Planned profile verification field

Planned profile response behavior:

- once implemented, `isVerified` should reflect persisted `ExpertProfile.IsVerified`
- current active backend does not yet persist this field

## 4. Expert Business + Expert APIs

Current active state:

- there is no active certificate endpoint yet for experts

Planned behavior:

- expert can create, list, view, update, and delete own certificates
- creating a certificate does not automatically mark the expert as verified
- verification changes only after admin review

### 4.1 Planned `POST /api/experts/me/certificates`

Purpose:

- create a certificate owned by the current expert

Auth:

- required
- role `Expert`

Request body:

```json
{
  "certificateName": "Clinical Toxicology Certificate",
  "issuingOrganization": "Vietnam Poison Control Academy",
  "issueDate": "2025-05-01T00:00:00Z",
  "expiryDate": "2028-05-01T00:00:00Z",
  "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001.pdf"
}
```

Success response example:

```json
{
  "statusCode": 201,
  "message": "Certificate created successfully.",
  "isSuccess": true,
  "data": {
    "id": "9f5d44f4-0e31-4931-983f-08f0fc8d15f1",
    "expertId": "7aaab7a4-6441-4f6c-88d0-e8451adf6d7b",
    "certificateName": "Clinical Toxicology Certificate",
    "issuingOrganization": "Vietnam Poison Control Academy",
    "issueDate": "2025-05-01T00:00:00Z",
    "expiryDate": "2028-05-01T00:00:00Z",
    "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001.pdf",
    "verificationStatus": "Pending",
    "rejectionReason": ""
  }
}
```

Client notes:

- `verificationStatus` should start as `Pending` unless business rules explicitly allow admin-side create with another initial state
- frontend should not infer verification from create success alone

### 4.2 Planned `GET /api/experts/me/certificates`

Purpose:

- list certificates owned by the current expert

Auth:

- required
- role `Expert`

Success response example:

```json
{
  "statusCode": 200,
  "message": "Success",
  "isSuccess": true,
  "data": [
    {
      "id": "9f5d44f4-0e31-4931-983f-08f0fc8d15f1",
      "expertId": "7aaab7a4-6441-4f6c-88d0-e8451adf6d7b",
      "certificateName": "Clinical Toxicology Certificate",
      "issuingOrganization": "Vietnam Poison Control Academy",
      "issueDate": "2025-05-01T00:00:00Z",
      "expiryDate": "2028-05-01T00:00:00Z",
      "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001.pdf",
      "verificationStatus": "Pending",
      "rejectionReason": ""
    }
  ]
}
```

### 4.3 Planned `GET /api/experts/me/certificates/{certificateId}`

Purpose:

- get detail for one certificate owned by the current expert

Auth:

- required
- role `Expert`

### 4.4 Planned `PUT /api/experts/me/certificates/{certificateId}`

Purpose:

- update one certificate owned by the current expert

Auth:

- required
- role `Expert`

Request body:

```json
{
  "certificateName": "Advanced Clinical Toxicology Certificate",
  "issuingOrganization": "Vietnam Poison Control Academy",
  "issueDate": "2025-05-01T00:00:00Z",
  "expiryDate": "2028-05-01T00:00:00Z",
  "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001-v2.pdf"
}
```

Client notes:

- if the chosen rule is to re-review changed files, expert updates may reset `verificationStatus` back to `Pending`
- this exact rule is still open in `expert-certs.hallucination.md`

### 4.5 Planned `DELETE /api/experts/me/certificates/{certificateId}`

Purpose:

- delete one certificate owned by the current expert

Auth:

- required
- role `Expert`

## 5. Admin Business + Admin APIs

Current active state:

- there is no active certificate endpoint yet for admins

Planned behavior:

- admin can manage certificates globally
- admin review controls certificate verification state
- expert verification state is derived from certificate review outcome

### 5.1 Planned `POST /api/admin/expert/certificates`

Purpose:

- create a certificate record for a target expert from the admin side

Auth:

- required
- role `Admin`

Request body:

```json
{
  "expertId": "7aaab7a4-6441-4f6c-88d0-e8451adf6d7b",
  "certificateName": "Clinical Toxicology Certificate",
  "issuingOrganization": "Vietnam Poison Control Academy",
  "issueDate": "2025-05-01T00:00:00Z",
  "expiryDate": "2028-05-01T00:00:00Z",
  "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001.pdf",
  "verificationStatus": "Pending",
  "rejectionReason": ""
}
```

Note:

- whether admin truly has full create authority is still an open policy risk and is tracked in `expert-certs.hallucination.md`

### 5.2 Planned `GET /api/admin/expert/certificates`

Purpose:

- list certificates across all experts

Auth:

- required
- role `Admin`

Example query usage:

- `/api/admin/expert/certificates`
- `/api/admin/expert/certificates?expertId=7aaab7a4-6441-4f6c-88d0-e8451adf6d7b`
- `/api/admin/expert/certificates?verificationStatus=Pending`

### 5.3 Planned `GET /api/admin/expert/certificates/{certificateId}`

Purpose:

- get detail for one certificate

Auth:

- required
- role `Admin`

### 5.4 Planned `PUT /api/admin/expert/certificates/{certificateId}`

Purpose:

- update certificate data and review state

Auth:

- required
- role `Admin`

Request body:

```json
{
  "certificateName": "Advanced Clinical Toxicology Certificate",
  "issuingOrganization": "Vietnam Poison Control Academy",
  "issueDate": "2025-05-01T00:00:00Z",
  "expiryDate": "2028-05-01T00:00:00Z",
  "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001-v2.pdf",
  "verificationStatus": "Verified",
  "rejectionReason": ""
}
```

Success response example:

```json
{
  "statusCode": 200,
  "message": "Certificate updated successfully.",
  "isSuccess": true,
  "data": {
    "id": "9f5d44f4-0e31-4931-983f-08f0fc8d15f1",
    "expertId": "7aaab7a4-6441-4f6c-88d0-e8451adf6d7b",
    "certificateName": "Advanced Clinical Toxicology Certificate",
    "issuingOrganization": "Vietnam Poison Control Academy",
    "issueDate": "2025-05-01T00:00:00Z",
    "expiryDate": "2028-05-01T00:00:00Z",
    "certificateUrl": "https://res.cloudinary.com/demo/raw/upload/v1/expert-certificates/cert-001-v2.pdf",
    "verificationStatus": "Verified",
    "rejectionReason": ""
  }
}
```

Client notes:

- when `verificationStatus` becomes `Verified`, the backend should recalculate `ExpertProfile.isVerified`
- when a verified certificate is later rejected, reset to pending, or deleted, the backend should recalculate again

### 5.5 Planned `DELETE /api/admin/expert/certificates/{certificateId}`

Purpose:

- delete a certificate globally from the admin side

Auth:

- required
- role `Admin`

## 6. Verified Endpoint List

Active endpoints as of 2026-04-21:

- none for expert certificates

Planned endpoints:

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

## 7. Changelog

- 2026-04-21: initialized planning use guide for `expert-certs`
- 2026-04-21: recorded verified current backend state and proposed expert/admin API contracts
