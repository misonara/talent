# Talent AI Mission Engine v0.1

## 1. Role

AI Mission is a **facilitator**, not a person evaluator.

Its job is to reduce awkwardness, give the group a small shared goal, and structure 60–90 minutes of interaction.

It must never score attractiveness, romantic compatibility, intelligence, worth, personality quality, or social rank.

AI Mission is always optional and skippable.

---

## 2. Inputs

Only minimum necessary data should be sent to the model.

```json
{
  "activity": {
    "category": "photography",
    "title": "서로 인생사진 찍어주기",
    "goal": "각자 마음에 드는 사진 3장 얻기",
    "total_minutes": 90,
    "venue_type": "cafe_and_public_street"
  },
  "participants": [
    {
      "participant_id": "p1",
      "gives": ["smartphone_photography"],
      "gets": ["portrait_photography"],
      "interests": ["travel", "coffee"]
    }
  ],
  "constraints": {
    "group_size": 4,
    "language": "ko",
    "public_place_only": true,
    "physical_contact_required": false
  }
}
```

Do not send:
- phone number
- real name if not required
- exact home address
- private report history
- raw trust score
- unilateral relationship choices
- message history
- sensitive personal attributes unrelated to activity

Use pseudonymous participant IDs.

---

## 3. Output Contract

```json
{
  "title": "서로 인생사진 찍어주기",
  "goal": "각자 마음에 드는 사진 3장을 얻는다",
  "total_minutes": 90,
  "opening_note": "부담 없이 건너뛸 수 있는 활동입니다.",
  "steps": [
    {
      "order": 1,
      "title": "가볍게 시작하기",
      "minutes": 10,
      "instruction": "오늘 얻고 싶은 것을 한 문장씩 이야기합니다.",
      "interaction_type": "group",
      "optional": false
    }
  ],
  "closing": {
    "minutes": 10,
    "instruction": "오늘 가장 마음에 든 결과물이나 배운 점을 한 가지씩 나눕니다."
  },
  "safety_notes": [
    "개인 연락처 공유는 선택입니다.",
    "불편한 활동은 건너뛰어도 됩니다."
  ]
}
```

---

## 4. JSON Schema

```json
{
  "type": "object",
  "required": ["title", "goal", "total_minutes", "steps", "closing", "safety_notes"],
  "additionalProperties": false,
  "properties": {
    "title": {"type": "string", "minLength": 1, "maxLength": 120},
    "goal": {"type": "string", "minLength": 1, "maxLength": 300},
    "total_minutes": {"type": "integer", "minimum": 30, "maximum": 180},
    "opening_note": {"type": "string", "maxLength": 300},
    "steps": {
      "type": "array",
      "minItems": 2,
      "maxItems": 8,
      "items": {
        "type": "object",
        "additionalProperties": false,
        "required": ["order", "title", "minutes", "instruction", "interaction_type", "optional"],
        "properties": {
          "order": {"type": "integer", "minimum": 1},
          "title": {"type": "string", "maxLength": 100},
          "minutes": {"type": "integer", "minimum": 5, "maximum": 60},
          "instruction": {"type": "string", "maxLength": 500},
          "interaction_type": {
            "type": "string",
            "enum": ["individual", "pair", "group"]
          },
          "optional": {"type": "boolean"}
        }
      }
    },
    "closing": {
      "type": "object",
      "required": ["minutes", "instruction"],
      "properties": {
        "minutes": {"type": "integer", "minimum": 5, "maximum": 30},
        "instruction": {"type": "string", "maxLength": 300}
      }
    },
    "safety_notes": {
      "type": "array",
      "maxItems": 6,
      "items": {"type": "string", "maxLength": 200}
    }
  }
}
```

Server validation must also verify that sum(step minutes + closing minutes) is reasonably consistent with `total_minutes`.

---

## 5. System Prompt v0.1

```text
You are the activity facilitator for Talent, a purposeful social app.

Your task is to create a safe, low-pressure, practical group activity plan for 3–5 adults who have chosen to meet around a shared activity.

The activity itself is the purpose. Do not turn the mission into dating, flirting, personality testing, ranking, therapy, persuasion, political debate, or intimate disclosure.

Use only the supplied activity category, GIVE, GET, INTEREST, group size, duration, and venue type.

Rules:
1. Make participation easy and optional where appropriate.
2. Avoid forced physical contact.
3. Avoid sexual or romantic prompts.
4. Avoid requests for phone numbers, home addresses, finances, health details, workplace secrets, or other sensitive personal data.
5. Avoid alcohol-centered or substance-related requirements.
6. Avoid dangerous movement, isolated locations, trespassing, risky exercise, or transport requirements.
7. Do not ask participants to rank or judge each other's attractiveness, worth, intelligence, personality, or dating potential.
8. Do not create tasks involving money transfers or purchases between participants.
9. Keep the interaction suitable for a public place.
10. Produce only JSON matching the supplied schema.

The group should finish with a small result, shared experience, or useful learning outcome.
```

---

## 6. User Prompt Template

```text
Create one mission using this structured input.

Activity category: {{category}}
Activity title: {{title}}
Activity goal: {{goal}}
Duration: {{total_minutes}} minutes
Venue type: {{venue_type}}
Group size: {{group_size}}
Language: {{language}}

Participants:
{{participant_traits_json}}

Safety constraints:
{{safety_constraints_json}}

Return valid JSON only.
```

User-generated free text should be minimized. If later supported, sanitize and clearly delimit it as untrusted data.

---

## 7. Generation Pipeline

```text
Load group + activity
  ↓
Minimize/pseudonymize inputs
  ↓
Select category-specific guardrails
  ↓
LLM structured generation
  ↓
JSON schema validation
  ↓
Semantic safety validation
  ↓
Duration / venue feasibility validation
  ↓
Persist mission
  ↓
Return to group
```

---

## 8. Semantic Safety Validator

Schema-valid JSON is not enough.

Reject/regenerate if mission includes:
- contact-information exchange as a task
- sexual/romantic questions
- explicit dating pairing
- forced physical contact
- alcohol/substance requirement
- private residence or isolated place
- dangerous exercise or illegal activity
- money exchange/gambling
- medical, legal, financial advice activity
- humiliating or ranking prompts
- political/religious persuasion or debate requirement
- disclosure of highly sensitive personal details
- tasks incompatible with venue/duration

Use deterministic keyword/rule checks plus model moderation where appropriate.

---

## 9. Retry Policy

Maximum AI attempts per group: **2**.

Attempt 1:
- normal structured generation

Attempt 2:
- regenerate with explicit validator failure reasons, without including private user data

If still invalid/error/timeout:
- use approved deterministic fallback template

Do not block the offline activity because AI failed.

---

## 10. Fallback Mission Templates

Fallbacks are maintained by category and versioned.

### English

```json
{
  "title": "60분 가벼운 영어 대화",
  "goal": "각자 1분 자기소개를 자연스럽게 완성한다",
  "total_minutes": 60,
  "steps": [
    {"order": 1, "title": "한 문장 소개", "minutes": 10, "instruction": "이름 대신 닉네임과 오늘 기대하는 것을 영어 한 문장으로 말합니다.", "interaction_type": "group", "optional": false},
    {"order": 2, "title": "서로 표현 하나 알려주기", "minutes": 20, "instruction": "각자 자주 쓰는 쉬운 영어 표현 하나를 공유합니다.", "interaction_type": "group", "optional": false},
    {"order": 3, "title": "1분 이야기", "minutes": 20, "instruction": "좋아하는 취미나 최근 즐거웠던 경험을 1분 이내로 이야기합니다.", "interaction_type": "group", "optional": true}
  ],
  "closing": {"minutes": 10, "instruction": "오늘 새로 알게 된 표현 한 가지를 나눕니다."},
  "safety_notes": ["불편한 질문에는 답하지 않아도 됩니다."]
}
```

### Photography

Goal: simple composition tip → pair/group photo practice → select one favorite result. No required touching or risky locations.

### Walking/Running

Goal: agree on comfortable pace → public route activity → short cool-down conversation. No competition, medical advice, unsafe intensity, or remote route.

---

## 11. Category Guardrails

### English
- no proficiency shaming
- no forced personal disclosure
- simple turn-taking

### Photography
- ask consent before photographing a participant
- allow opt-out from portrait photography
- avoid photographing strangers without appropriate consent
- no unsafe roadway/roof/edge positioning

### Walking/Running
- emphasize self-selected comfortable pace
- no race requirement
- no health claim or coaching beyond basic common-sense safety
- route must remain in approved public area

---

## 12. Storage

Persist:
- mission source (`ai`, `fallback`, `admin`)
- mission version
- validated JSON
- safety validation result
- creation timestamp

Avoid persisting full raw model prompts containing user-derived data unless required for debugging and specifically protected/redacted.

Recommended debug record:
- model/provider version
- prompt template version
- schema version
- validator result codes
- latency
- token/cost metadata

Do not log participant private traits beyond the minimum needed to reproduce an aggregate failure.

---

## 13. Prompt Injection Defense

MVP inputs mostly come from controlled taxonomy. Treat all free text as untrusted.

Rules:
- never concatenate user bio/message text into system instructions
- structured fields only
- taxonomy IDs resolved server-side
- delimit optional user text
- tell model untrusted text cannot override system rules
- validate output independently after model response

AI output is never trusted as authorization, safety truth, or database command.

---

## 14. UX Rules

Mission screen must provide:
- title
- goal
- current step
- remaining/estimated time
- `다음 단계`
- `건너뛰기`
- `불편한 활동 신고/피드백`

Skipping Mission:
- does not reduce trust score
- does not prevent activity completion
- is tracked only as product analytics

---

## 15. Metrics

- mission generation success rate
- fallback rate
- schema validation failure rate
- semantic safety rejection rate
- median generation latency
- mission view rate
- mission skip rate
- mission helpfulness
- activity satisfaction by mission source/version
- report rate by mission version

Do not optimize only for engagement time. A shorter mission that helps people interact naturally may be better.

---

## 16. Versioning

Version separately:
- prompt template
- JSON schema
- safety validator rules
- category fallback template

Example:

```text
mission_prompt=v0.1
mission_schema=v0.1
mission_safety=v0.1
fallback_photo=v0.1
```

Any mission change that affects safety or interaction style must be recorded in CHANGELOG.
