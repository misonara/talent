# Talent Security & Privacy Architecture v0.1

## 1. Security Goal

Talent는 온라인 정보 서비스가 아니라 **낯선 사용자를 실제 오프라인 만남으로 연결하는 서비스**다. 따라서 보안 목표는 단순 계정 보호를 넘어 다음을 포함한다.

1. 계정 도용 방지
2. 불필요한 개인정보 노출 최소화
3. 차단/신고 사용자 간 재노출 방지
4. 일방적 관계 선택 비밀 유지
5. 안전하지 않은 그룹/활동 생성 방지
6. 채팅 악용과 스팸 제한
7. 운영자 접근의 감사 가능성
8. AI가 권한 또는 안전 정책을 우회하지 못하게 하기

Safety와 Security를 별도 부가기능으로 취급하지 않는다.

---

## 2. Data Classification

### Restricted
가장 강한 접근 통제가 필요하다.

- 인증 provider subject / phone-derived identifier
- 신고 본문 및 moderation 메타데이터
- trust_events 세부 내역
- directional relationship_choices
- suspension/restriction reason
- admin audit log

### Group-confidential
확정된 동일 그룹 구성원에게만 제한적으로 공개한다.

- 닉네임
- age band
- 제한된 profile image
- GIVE / GET / INTEREST
- 활동 참여 정보

### User-private
본인과 서버/필요 운영자만 접근한다.

- 설정된 활동 가능 영역 전체
- 상세 availability
- 자신의 review text
- 자신의 saved relationship choice

### Public/product content
- taxonomy
- 공개 활동 카드
- 공개 활동 목표/일정/대략 지역

---

## 3. Authentication

MVP 원칙:

- phone/provider verification은 검증된 외부 auth provider 사용 권장
- OTP 직접 저장 금지
- raw phone number를 일반 application query에서 사용하지 않음
- access token은 짧은 수명
- refresh/session revoke 지원
- suspended/deleted 상태는 토큰이 유효해도 API에서 재검증

> **Design Decision Required:** Firebase Auth, Supabase Auth, 별도 한국 휴대폰 인증 사업자 조합 중 최종 선택 필요.

---

## 4. Authorization Model

서버는 모든 요청에서 resource-level authorization을 수행한다.

### Example rules

`GET /groups/{id}`
- 해당 그룹 멤버 또는 admin만 허용

`GET /conversations/{id}/messages`
- active conversation member만 허용

`PUT relationship-choice`
- completed group의 실제 두 멤버 사이에서만 허용

`GET reports/{id}`
- reporter에게도 전체 moderation 내부 상태를 노출하지 않고 필요한 사용자-facing 상태만 제공

클라이언트가 보내는 `user_id`, `role`, `verified`, `trust` 값을 신뢰하지 않는다.

---

## 5. IDOR Prevention

UUID를 사용해도 authorization이 없으면 IDOR는 해결되지 않는다.

모든 object fetch는 다음 패턴을 사용한다.

```text
load resource
→ verify requester relationship/ownership
→ apply block/privacy rules
→ serialize allowed fields only
```

권한 없는 private resource에는 가능한 경우 `404`를 반환해 존재 여부를 숨긴다.

테스트 필수 대상:
- 다른 사용자의 group
- 다른 사용자의 review
- 다른 사용자의 relationship choice
- 다른 conversation/message
- report/moderation data

---

## 6. Relationship Choice Confidentiality

가장 중요한 privacy invariant 중 하나다.

- A → B `personal_interest`
- B가 reciprocal choice를 하지 않으면 B는 A의 선택 사실을 알 수 없다.

금지:
- "누가 나를 선택했는지" API
- unilateral count badge
- analytics/event payload를 client에 노출
- push notification으로 상대 선택 암시

Mutual connection creation은 server transaction 내부에서만 판정한다.

Logs에서도 raw directional choice를 일반 analytics로 보내지 않는다.

---

## 7. Block Invariant

A가 B를 block하면 즉시 다음을 보장해야 한다.

- future automatic matching pair exclusion
- 새로운 mutual connection 생성 금지
- direct conversation interaction disable
- 사용자 검색/추천에서 가능한 범위 내 상호 숨김
- block 사실 자체는 상대에게 명시적으로 통보하지 않음

Matching에서는 directional block 두 방향을 모두 검사한다.

```text
blocked(A,B) OR blocked(B,A) => same group forbidden
```

Block removal이 과거 connection을 자동 복구하지는 않는다.

---

## 8. Location Privacy

MVP는 상시 위치 추적을 사용하지 않는다.

수집 원칙:
- broad activity area
- approved venue location
- check-in 시 필요한 경우 coarse area only

금지:
- exact home address
- background continuous GPS
- 다른 사용자에게 실시간 좌표 공개
- 장소 도착 전 불필요한 상세 이동 경로 공개

Venue는 공개 상업/공공 장소 중심으로 운영한다.

---

## 9. Chat Abuse Controls

기본 통제:
- group chat은 group confirmation 후 활성화
- direct chat은 mutual connection 이후에만 활성화
- block 시 direct sending 즉시 금지
- per-user/per-conversation rate limit
- message length limit
- repeated identical message / link spam detection
- report entry point 제공

MVP에서는 메시지 body를 product analytics에 넣지 않는다.

향후 URL/image attachment를 지원하면 별도 malware/phishing/content moderation 설계가 필요하다.

---

## 10. Report Confidentiality

Report data는 피신고 사용자에게 공개하지 않는다.

Reporter에게 제공 가능한 정보:
- 접수됨
- 검토 중
- 필요한 조치 완료 등 제한된 상태

공개하면 안 되는 정보:
- 내부 moderator notes
- 다른 신고자의 존재/신원
- 내부 risk score
- 조사 근거 전체

Severe report는 운영 검토 queue 우선순위를 높일 수 있으나, 자동으로 위반 확정을 의미하지 않는다.

---

## 11. Trust System Safety

`trust_events`는 내부 reliability/safety ledger다.

원칙:
- raw score 사용자 공개 금지
- 설명 가능한 event 기반
- new user neutral baseline
- report는 `report_upheld`와 단순 `report_created`를 구분
- admin adjustment는 reason/audit 기록

사용자 UI는 예:
- 본인인증 완료
- 활동 참여 경험 있음
- 최근 약속 이행 상태 양호

처럼 제한된 indicator만 제공한다.

---

## 12. AI Security

LLM은 절대 다음의 source of truth가 아니다.

- authorization
- block state
- safety approval
- trust status
- group membership
- payment/account state

Prompt injection 대응:
- controlled taxonomy 중심 input
- user free text 최소화
- free text는 untrusted data로 delimiter 처리
- system prompt와 user content 분리
- structured output schema validation
- semantic safety validator
- AI output을 SQL/API command로 직접 실행하지 않음

AI provider에 전송하는 데이터는 최소화하고 pseudonymize한다.

---

## 13. Account Enumeration

Auth/recovery flow에서 다음과 같은 차이를 최소화한다.

금지 예:
- "이 전화번호는 가입되어 있습니다"를 공격자가 대량 확인 가능

대응:
- generic response
- strict auth rate limiting
- provider-level abuse protection
- IP/device/user bucket 조합

---

## 14. Rate Limiting

별도 bucket 권장:

### Very strict
- auth/session verification
- report submission

### Strict
- activity join/cancel burst
- profile nickname/photo update

### Moderate
- chat send
- activity reads

Rate limit 결과는 `429` + retry metadata를 제공하되 내부 anti-abuse threshold를 과도하게 노출하지 않는다.

---

## 15. Logging

Application logs에 넣지 말아야 할 데이터:
- raw phone
- OTP
- auth token
- report description
- message body
- exact location
- unilateral relationship choice details

로그 권장:
- request_id
- pseudonymous user id
- endpoint
- result/status
- latency
- security decision code

Admin action은 별도 immutable/auditable log를 유지한다.

---

## 16. Secrets

- `.env` commit 금지
- provider/API keys는 secret manager 사용
- production/dev key 분리
- least privilege
- key rotation process 문서화

Repository에는 `.env.example`만 둔다.

---

## 17. Database Protections

- unique constraints를 business invariant의 최종 방어선으로 사용
- foreign key 사용
- relationship/group confirmation transaction 적용
- production backup 암호화
- DB role least privilege
- admin query 접근 제한

특히 다음은 concurrency test가 필요하다.
- 동일 사용자의 중복 application
- overlapping group confirmation
- reciprocal relationship choices 동시 제출
- mutual connection 중복 생성
- double check-in

---

## 18. Moderation Admin Security

Minimal Admin이라도 일반 user app보다 강한 통제가 필요하다.

필수:
- admin role separation
- MFA 권장/필수
- sensitive record access audit
- suspension action audit
- report resolution audit
- bulk export 제한

관리자가 unilateral relationship choice를 일상적으로 조회할 수 있는 UI는 만들지 않는다.

---

## 19. Abuse Cases to Test

1. 차단한 사용자가 다른 account로 반복 접근
2. 상대 group UUID 추측/획득 후 profile 조회
3. direct chat URL을 조작해 mutual connection 없이 메시지 전송
4. repeated join/cancel로 그룹 운영 방해
5. 허위 대량 신고
6. chat spam/link spam
7. AI input에 prompt injection 포함
8. check-in API 반복 호출
9. reciprocal choice race로 중복 conversation 생성
10. admin endpoint 일반 user 접근

---

## 20. Security Test Checklist Before Closed Beta

- [ ] auth rate-limit test
- [ ] IDOR tests for all private resources
- [ ] block invariant integration test
- [ ] unilateral relationship privacy test
- [ ] group membership authorization test
- [ ] direct-chat mutual requirement test
- [ ] report confidentiality test
- [ ] duplicate/race transaction tests
- [ ] AI mission schema + safety rejection tests
- [ ] log PII review
- [ ] admin authorization/audit review
- [ ] dependency vulnerability scan
- [ ] secret scanning enabled

---

## 21. Incident Minimum Process

Closed Beta 전 최소한 다음이 있어야 한다.

1. safety/security report intake
2. account restriction capability
3. evidence/log preservation path
4. internal severity classification
5. user-facing response policy
6. post-incident review

긴급 위험 상황에 대해 앱이 공공 긴급 서비스 자체를 대체한다고 표현해서는 안 된다.

---

## 22. Open Decisions

### Design Decision Required — Auth provider
Firebase Auth / Supabase Auth / Korea phone verification integration 비교 필요.

### Design Decision Required — Data retention
Messages, reports, deleted-account data, moderation evidence의 정확한 retention period는 개인정보 정책 및 국내 법률 검토와 함께 확정해야 한다.

### Design Decision Required — Profile photo moderation
MVP에서 automated moderation provider를 쓸지 admin review만 할지 결정 필요.
