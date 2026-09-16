# Changelog

## 2026-09-16

### Added
- Initialized Purposeful Social App project repository.
- Added core product vision and principles.
- Added MVP development specification prompt.
- Added phased product and development roadmap.
- Added `PRODUCT_SPEC.md` v0.1.
- Added `ERD.md` v0.1.
- Added `API_SPEC.md` v0.1 covering auth, profile, activity, groups, check-in, reviews, relationship choices, chat, reports/blocks, admin, idempotency and concurrency.
- Added `MATCHING_ENGINE.md` v0.1 with explainable 3–5 person rule-based group matching and Cold Start fallback rules.
- Added `AI_MISSION.md` v0.1 with structured JSON output, prompt template, safety validation, retry/fallback policy and category guardrails.
- Added `SECURITY.md` v0.1 covering authorization, IDOR, relationship-choice confidentiality, blocks, location privacy, chat abuse, reports, AI security and Closed Beta security checks.

### Decisions
- Frozen the first MVP at 15 core capabilities.
- Default group size is 4, with 3–5 allowed.
- Closed Beta categories are English, photography, and walking/running.
- User-created public activities are deferred beyond MVP.
- Deposits, payments, subscriptions, and venue monetization are deferred beyond MVP.
- Group chat is available after group confirmation; unrestricted 1:1 DM before activity completion is excluded.
- AI Mission is included in MVP as a guided facilitator, not a person-rating system.
- Relationship choices are private and directional; direct chat opens only after a policy-compatible mutual connection.
- Mutual relationship compatibility v0.1 requires matching/compatible intent: personal↔personal, project↔project, group-again/another-activity combinations for activity-oriented connection.
- Report/block and minimal admin moderation are MVP requirements.
- Matching diversity in MVP is based on GIVE/GET/interest coverage, not demographic or attractiveness scoring.
- A 2-person auto-formed group is not allowed in MVP; minimum remains 3.
- AI Mission failure never blocks the activity; approved fallback missions are required.

### Design Decisions Still Open
- Final authentication/phone verification provider.
- Exact data-retention periods for messages/reports/deleted accounts/moderation evidence.
- Profile photo moderation approach.
- Final implementation stack.

### Current Focus
- Select technology stack and deployment architecture.
- Define repository/application structure.
- Define analytics event schema.
- Create Sprint 0 engineering backlog.
- Prepare clickable prototype and Closed Beta operations plan.
