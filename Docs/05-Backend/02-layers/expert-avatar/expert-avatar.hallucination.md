---
doc_role: decision-log
module: expert-avatar
kind: response-contract-amendment
doc_type: hallucination
status: resolved
last_updated: 2026-05-03
owners: [backend-team]
verification_status: decisions-recorded
---

# Expert Avatar Hallucination Log

This file keeps decision history. Resolved decisions stay here because this module already has backend implementation and this task is now an amendment, not a fresh implementation.

## Open Risks

None.

## Closed Risks

### Decision 1. Initial Avatar Field For `GET /api/experts/me/consultations`

Status: `Superseded`

Original decision date: 2026-05-02

Original options:

1. Add `expertAvatarUrl` only.
2. Add `expertId`, `expertName`, and `expertAvatarUrl`.
3. Add `userAvatarUrl` for the other participant and no expert avatar.
4. Add both expert and participant display fields.

Original decision:

- Option 1 was selected.
- Backend implemented nullable `ExpertAvatarUrl` on `ExpertConsultationResponse`.
- Mapping reads the authenticated expert account avatar.

Superseding note:

- This implementation is removed by the latest decision.
- New frontend decision requires participant avatar only for expert screen.

### Decision 2. Frontend Clarification

Status: `Closed`

Decision date: 2026-05-03

Decision:

- `GET /api/users/me/consultations` keeps `expertAvatarUrl`.
- `GET /api/experts/me/consultations` must provide the other participant avatar.
- Implement this as additive `userAvatarUrl` on `ExpertConsultationResponse`.
- Remove implemented `expertAvatarUrl` from `ExpertConsultationResponse`.

Reason:

- member screen displays expert info.
- expert screen displays member/rescuer info.
- changing docs as an amendment reduces risk that future agents overwrite or miss already implemented avatar work.

### Decision 3. Remove Expert Self Avatar From Expert History

Status: `Closed`

Decision date: 2026-05-03

Decision:

- `GET /api/experts/me/consultations` should keep only `userAvatarUrl`.
- Remove `expertAvatarUrl` from `ExpertConsultationResponse`.
- Remove expert-history mapping/tests for authenticated expert avatar.

Reason:

- frontend explicitly needs member/rescuer avatar on expert screen.
- authenticated expert's own avatar is not needed in this endpoint.
