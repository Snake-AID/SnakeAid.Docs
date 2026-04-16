# LiveKit Domain Context

This analysis file preserves the original pre-consultation investigation that justified choosing LiveKit Cloud before the rest of the consultation flow was implemented.

## Why it came first

The team needed to validate the video-call technology decision before investing in booking, payment, and room lifecycle details. Remote expert consultation depends on reliable real-time media, so provider feasibility had to be confirmed first.

## Core conclusions

- SnakeAid consultation requires a managed real-time video provider for patient/rescuer to expert calls.
- LiveKit Cloud fit the architecture because it offers managed SFU infrastructure, a mature Flutter client SDK, and a standard JWT-based auth model.
- Backend ownership is limited to integration concerns:
  - issue short-lived join tokens
  - manage room lifecycle through provider APIs
  - validate provider webhook events
- WebRTC media transport itself remains outside the consultation backend module.

## Business framing captured by the original standalone docs

- video call sits inside the consultation lifecycle, not beside it
- room creation is only valid after the consultation reaches an allowed paid/ready state
- room identity follows consultation identity
- expert role needs broader publish permissions than member/rescuer roles

## Historical note

This content was merged from the previous standalone `live-kit-cloud` module on 2026-04-12 so the P3 consultation flow can present LiveKit as Operation 01 instead of as a sibling module.
