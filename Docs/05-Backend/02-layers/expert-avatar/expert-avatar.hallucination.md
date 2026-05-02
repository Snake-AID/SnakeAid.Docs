---
doc_role: decision-log
module: expert-avatar
kind: response-contract
doc_type: hallucination
status: resolved
last_updated: 2026-05-03
owners: [backend-team]
verification_status: decisions-recorded
---

# Expert Avatar Hallucination Log

This file tracks decisions that need user/business input. When a risk is solved, keep the original option list, record the decision, merge the decision into baseline docs, then mark the risk closed.

## Open Risks

None.

## Closed Risks

### Risk 1. Meaning Of Avatar On `GET /api/experts/me/consultations`

Status: `Closed`

Why this needed decision:

- Business request said: add avatar to `expert/me/consultation`.
- Code route is `GET /api/experts/me/consultations`.
- Current response DTO is `ExpertConsultationResponse`.
- Current DTO exposes the other participant as `UserId` and `UserName`.
- It does not expose `ExpertId` or `ExpertName`, because the authenticated actor is already the expert.

Original options:

1. Add `expertAvatarUrl` only.
   - Pros: directly satisfies wording "expert avatar".
   - Cons: response still has no `expertId` or `expertName`; mobile may need to infer the expert is the authenticated user.

2. Add `expertId`, `expertName`, and `expertAvatarUrl`.
   - Pros: every screen gets complete expert display identity from the row itself.
   - Cons: duplicates current authenticated expert data on every item.

3. Add participant-specific fields: `userAvatarUrl` for the other participant, and no expert avatar.
   - Pros: matches current DTO shape, where `UserId/UserName` refer to caller/rescuer.
   - Cons: does not satisfy the literal "expert avatar" goal.

4. Add both expert and participant display fields.
   - Pros: most explicit; future screens do not infer identity.
   - Cons: largest contract expansion.

Decision record:

- Decision date: 2026-05-02
- Decision maker: user
- Selected option: Option 1
- Final decision: for `GET /api/experts/me/consultations`, add `expertAvatarUrl` only.
- Scope note: do not add `expertId`, `expertName`, `userAvatarUrl`, or participant avatar fields in this round.
- Baseline docs updated: introduction, roadmap, sourcecode, useguide.
