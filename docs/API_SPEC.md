# Talent REST API Specification v0.1

## 1. Conventions

- Base path: `/api/v1`
- Transport: HTTPS only
- Payload: JSON UTF-8
- Authentication: Bearer access token issued by the selected auth provider
- User identifiers exposed to clients: UUID
- Server timestamps: ISO-8601 UTC
- Pagination: cursor-based for activities/messages
- Error envelope:

```json
{
  "error": {
    "code": "ACTIVITY_FULL",
    "message": "The activity is no longer accepting applications.",
    "request_id": "req_..."
  }
}
```

### Standard status codes

- `200` success
- `201` created
- `204` success without body
- `400` malformed request
- `401` unauthenticated
- `403` authenticated but unauthorized
- `404` resource not found or deliberately concealed
- `409` state/conflict/idempotency conflict
- `422` valid JSON but invalid business rule
- `429` rate limited

### Security principle

Return `404` rather than revealing the existence of privacy-sensitive resources when the requester has no membership/ownership right.

---

## 2. Authentication & Account

> **Design Decision Required:** final phone OTP/Auth vendor is not yet selected. The API assumes authentication is completed through a provider and the backend receives a verified provider identity. Raw OTP codes must never be stored by the application DB.

### POST `/auth/session`

Purpose: create/update the local user after successful provider authentication.

Auth: verified provider token.

Request:

```json
{
  "provider_subject": "opaque-provider-id",
  "birth_year": 1992,
  "terms_version": "2026-09-16",
  "privacy_version": "2026-09-16",
  "safety_policy_version": "2026-09-16"
}
```

Response `201`:

```json
{
  "user_id": "uuid",
  "account_status": "active",
  "onboarding_state": "profile_required"
}
```

Rules:
- minimum-age policy enforced server-side
- derive/store only a protected phone identifier or provider identity mapping
- never accept client-provided `verified_at`

Idempotency: provider subject acts as natural idempotency key.

Rate limit: strict.

### GET `/me`

Purpose: return own account/onboarding state.

Response includes account status, verification state, profile completeness, trait counts, configured areas and availability summary.

### DELETE `/me`

Purpose: request account deletion/deactivation.

Response: `204`.

Rules: legal retention for reports/moderation data may outlive user-facing deletion where policy/law requires.

---

## 3. Profile & Onboarding

### PUT `/me/profile`

Request:

```json
{
  "nickname": "민수",
  "bio": "사진과 산책을 좋아합니다.",
  "occupation_category": "design",
  "profile_photo_url": "https://...",
  "age_band": "30-34"
}
```

Rules:
- client cannot set verification/trust fields
- nickname and image moderation required
- age band must be derived/validated from account birth year rather than freely forged

### PUT `/me/traits`

Purpose: replace one trait category atomically.

Request:

```json
{
  "trait_type": "give",
  "items": [
    {"taxonomy_item_id": "uuid", "self_level": 3},
    {"taxonomy_item_id": "uuid", "self_level": 2}
  ]
}
```

Limits:
- GIVE <= 5
- GET <= 5
- INTEREST <= 10

Transaction: yes, replace old set + insert new set.

### GET `/taxonomy?type=skill|interest|activity_category`

Purpose: return active controlled vocabulary.

Public/authenticated read. Cacheable.

### PUT `/me/availability`

Request:

```json
{
  "slots": [
    {"weekday": 5, "start_minute": 1140, "end_minute": 1320}
  ]
}
```

Validation: no invalid overlaps, start < end, max reasonable number of slots.

### PUT `/me/activity-areas`

Request:

```json
{
  "areas": [
    {"area_code": "AREA_CODE", "label": "활동 지역명"}
  ]
}
```

Privacy: accepts approved broad activity areas only; no home address or arbitrary exact coordinate.

---

## 4. Activity Discovery

### GET `/activities`

Query:
- `cursor`
- `limit` max 50
- `category_id`
- `area_code`
- `from`
- `to`

Returns activity cards only. It does not expose applicant identities.

Response:

```json
{
  "items": [
    {
      "id": "uuid",
      "title": "영어 못해도 영어로 60분",
      "category": {"id": "uuid", "name": "영어"},
      "starts_at": "2026-09-19T11:00:00Z",
      "ends_at": "2026-09-19T12:30:00Z",
      "area": {"code": "AREA_CODE", "label": "활동 지역명"},
      "target_participants": 4,
      "goal": "1분 자기소개 완성",
      "application_state": "not_applied"
    }
  ],
  "next_cursor": null
}
```

### GET `/activities/{activityId}`

Returns details, approved venue summary if visible, application deadline/state, and user's own application state.

Do not expose the full applicant list before group confirmation.

### POST `/activities/{activityId}/applications`

Purpose: join/apply.

Headers: `Idempotency-Key` required.

Request:

```json
{
  "acknowledge_rules": true
}
```

Server checks:
- active/verified account
- onboarding minimum complete
- activity open
- not already selected into overlapping confirmed activity
- not under participation restriction

Response `201`:

```json
{
  "application_id": "uuid",
  "status": "applied"
}
```

Transaction: yes.

### DELETE `/activities/{activityId}/applications/me`

Purpose: cancel own application.

Rules:
- if group already confirmed, apply cancellation policy and record late-cancel trust event when applicable
- cancellation is idempotent

---

## 5. Groups

### GET `/groups/{groupId}`

Authorization: confirmed group member or authorized admin only.

Returns:
- activity summary
- venue summary
- start/end
- confirmed members' limited profiles
- user's attendance state
- mission availability
- group conversation id

Member profile exposure is limited to nickname, age band, profile image, GIVE/GET/INTEREST and safe trust indicators.

### GET `/me/groups?state=upcoming|past`

Returns user's own groups only.

### POST `/groups/{groupId}/checkins`

Purpose: check in.

Headers: `Idempotency-Key` required.

Request:

```json
{
  "method": "app",
  "coarse_area_code": "AREA_CODE"
}
```

Rules:
- member only
- valid time window
- no continuous GPS requirement
- coarse area is optional and must not be retained beyond stated attendance purpose

Response `201` or existing record `200`.

Transaction: update `checkins`, `group_members.attendance_status`, and attendance/trust ledger consistently.

### POST `/groups/{groupId}/complete`

MVP: admin/system endpoint, not arbitrary member endpoint.

Purpose: mark eligible active group completed and unlock reviews/relationship choices.

---

## 6. AI Mission

### GET `/groups/{groupId}/mission`

Authorization: group member/admin.

Behavior:
1. return existing safety-validated mission if present
2. otherwise generation occurs server-side through the Mission Engine
3. if AI generation/validation fails, persist and return fallback mission

Response:

```json
{
  "id": "uuid",
  "source": "ai",
  "title": "서로 인생사진 찍어주기",
  "goal": "각자 마음에 드는 사진을 얻는다",
  "total_minutes": 90,
  "steps": [],
  "skippable": true
}
```

Clients never send raw system prompts or arbitrary safety policy text.

### POST `/groups/{groupId}/mission/skip`

Purpose: analytics signal only; skipping must not penalize trust/reliability.

---

## 7. Completion & Reviews

### PUT `/groups/{groupId}/review`

Precondition: group completed and requester was a member.

Request:

```json
{
  "satisfaction": 4,
  "mission_helpfulness": 3,
  "would_join_similar": true,
  "safety_issue": false,
  "feedback_text": "진행이 자연스러웠습니다."
}
```

Rules:
- one private review per member/group; update allowed for limited window
- never expose member-level ratings publicly
- `safety_issue=true` should surface a safe path to report but must not silently create a misconduct allegation without user's confirmation/details

### PUT `/groups/{groupId}/relationship-choices/{targetUserId}`

Preconditions:
- group completed
- requester and target both belonged to same group
- requester != target
- pair not blocked either direction

Request:

```json
{
  "choice_type": "personal_interest"
}
```

Allowed: `group_again`, `another_activity`, `project`, `personal_interest`, `none`.

Privacy:
- response returns only requester's saved choice
- never reveals reciprocal choice directly
- notification is emitted only if a mutual connection is created

Transaction:
1. upsert directional choice
2. read reciprocal choice with lock/serializable-safe pattern
3. evaluate compatibility matrix
4. idempotently create mutual connection
5. idempotently create direct conversation
6. emit post-commit mutual notification

Compatibility v0.1:
- `personal_interest` + `personal_interest` -> `personal`
- `project` + `project` -> `project`
- `group_again` + `group_again` -> `group_again`
- `another_activity` + `another_activity` -> `activity`
- `group_again` + `another_activity` (either direction) -> `activity`
- all other mixed combinations -> no connection yet

> **Design Decision Required:** if user research shows users expect broader cross-intent matching, revise the compatibility matrix explicitly rather than inferring silently.

### GET `/connections`

Returns only active mutual connections belonging to requester.

No endpoint exists for "who liked me" or unilateral choices.

---

## 8. Chat

### GET `/conversations`

Returns only conversations where requester is an active member.

### GET `/conversations/{conversationId}/messages?cursor=...&limit=...`

Authorization: active conversation member.

Ordering: newest page or documented stable ordering; cursor required for history.

### POST `/conversations/{conversationId}/messages`

Request:

```json
{
  "body": "안녕하세요!"
}
```

Rules:
- sender must be active member
- direct conversation requires active mutual connection and neither party blocked
- length/content limits
- anti-spam throttling
- moderation/report hooks

Rate limit: per user + conversation.

### DELETE `/messages/{messageId}`

MVP: soft-delete own message within policy; admin moderation can hide separately in admin layer.

---

## 9. Safety

### POST `/blocks`

Request:

```json
{"blocked_user_id": "uuid"}
```

Transaction/effects:
- create block idempotently
- prevent future mutual connection creation
- disable direct interaction for that pair
- exclude pair from future grouping
- do not reveal blocker identity to blocked user

### DELETE `/blocks/{blockedUserId}`

Unblock does not automatically restore ended/disabled conversations.

### POST `/reports`

Request:

```json
{
  "reported_user_id": "uuid",
  "group_id": "uuid",
  "category": "harassment",
  "description": "private report text"
}
```

Rules:
- reporter must have legitimate context when `group_id` supplied
- protect report details from reported user
- duplicate/spam protections
- severe categories can trigger immediate safety workflow but not automatic guilt determination

### GET `/me/trust-indicators`

Returns UI-safe indicators only, e.g.:

```json
{
  "verified": true,
  "activity_count_band": "5-9",
  "reliability_label": "high"
}
```

Never return raw trust ledger weights or internal score.

---

## 10. Minimal Admin API

All endpoints require admin role, audit logging and least privilege.

- `GET /admin/reports`
- `GET /admin/reports/{id}`
- `PATCH /admin/reports/{id}`
- `GET /admin/users/{id}`
- `POST /admin/users/{id}/suspend`
- `POST /admin/users/{id}/restore`
- `GET /admin/activities`
- `POST /admin/activities`
- `PATCH /admin/activities/{id}`

Admin endpoints must not expose unilateral relationship choices unless strictly necessary for an investigated safety event and access is audited.

---

## 11. Idempotency & Concurrency Requirements

Idempotency required for:
- activity apply
- check-in
- relationship choice writes
- mutual connection creation
- conversation creation
- block creation

Concurrency-critical transactions:
- selecting applications into groups
- confirming groups
- preventing overlapping confirmed attendance
- mutual connection creation
- trust-event creation from attendance

Recommended DB mechanisms:
- unique constraints as final protection
- row/advisory locks for group confirmation
- transaction isolation appropriate to selection workload
- outbox pattern for notifications after committed state

---

## 12. Rate Limit Baseline

Exact numbers are deployment configuration, not API contract. Separate buckets should exist for:
- auth/session attempts
- activity applications/cancellations
- chat messages
- reports
- profile edits

Rate limits must be tighter for unauthenticated/auth endpoints and abuse-sensitive endpoints.

---

## 13. Analytics Events Triggered by API State Changes

- `signup_complete`
- `profile_complete`
- `activity_view`
- `activity_join`
- `activity_cancel`
- `group_confirmed`
- `checkin`
- `mission_view`
- `mission_skip`
- `activity_complete`
- `review_submit`
- `relationship_choice_submit`
- `mutual_connection`
- `second_meeting`
- `report_submit`

Analytics events must use pseudonymous user identifiers and exclude raw report text, phone identifiers, message bodies and precise location.
