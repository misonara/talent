# MVP Development Specification Prompt

아래 프롬프트는 이 프로젝트를 실제 MVP 개발 직전 수준까지 구체화하기 위한 기준 프롬프트다.

---

앞서 기획한 **Purposeful Social App / 목적형 관계 플랫폼**을 실제 개발 가능한 수준으로 설계하라.

서비스의 핵심 철학은 다음과 같다.

> 사람을 찾지 말고, 같이 할 일을 찾는다.

이 앱은 소개팅 앱이 아니다. 또한 단순 취미 모임 앱도 아니다.

사용자가 자신의 **GIVE(내가 나눌 수 있는 재능/능력)**, **GET(배우고 싶은 것)**, **INTEREST(함께하고 싶은 활동)**를 기반으로 3~5명의 소규모 오프라인 활동에 참여하고, 그 과정에서 친구, 네트워크, 프로젝트 파트너, 취미 동료 또는 개인적인 인연이 자연스럽게 형성되도록 하는 것이 목적이다.

연애는 서비스의 직접 목적이 아니라 활동 이후 발생할 수 있는 관계 중 하나다.

## 1. MVP 범위 확정

첫 버전에서 반드시 필요한 기능을 **15개 이내**로 제한하라.

다음 후보를 `MVP 필수`, `MVP 선택`, `Phase 2`, `Phase 3`으로 분류하고 이유를 설명하라.

- 회원가입
- 휴대폰 본인인증
- 프로필
- 프로필 사진
- GIVE
- GET
- INTEREST
- 활동 가능 시간
- 활동 가능 지역
- 활동 추천
- 활동 검색
- 활동 참가
- 모임 생성
- AI 그룹 매칭
- 그룹 채팅
- 개인 채팅
- AI Mission
- 체크인
- 모임 후기
- 관계 선택
- Mutual Match
- Trust Profile
- 신고
- 차단
- 참가 보증금
- 결제
- 구독
- Venue 제휴

MVP의 핵심 가설은 다음 질문을 검증하는 것이다.

> 사용자가 실제로 낯선 사람과 안전하게 만나고, 함께 활동하고, 다시 만나고 싶다고 느끼는가?

## 2. Database / ERD 설계

실제 개발 가능한 관계형 데이터베이스 구조를 설계하라.

최소 다음 Entity를 검토하라.

- users
- profiles
- skills
- user_gives
- user_gets
- interests
- user_interests
- availability
- user_locations
- activities
- activity_categories
- activity_participants
- participation_requests
- groups
- group_members
- missions
- checkins
- reviews
- relationship_choices
- mutual_connections
- conversations
- conversation_members
- messages
- trust_events
- reports
- blocks
- venues
- payments
- notifications

각 테이블에 대해 다음을 작성하라.

- 필드명
- 데이터 타입
- PK/FK
- null 허용 여부
- index
- unique constraint
- 주요 관계
- 개인정보 여부

ERD를 Mermaid 문법으로도 작성하라.

## 3. API 설계

REST API 기준으로 실제 개발자가 사용할 수 있는 명세를 작성하라.

예시:

- `POST /auth/signup`
- `GET /me`
- `PUT /me/profile`
- `PUT /me/gives`
- `PUT /me/gets`
- `GET /activities`
- `GET /activities/{id}`
- `POST /activities/{id}/join`
- `POST /groups/{id}/checkin`
- `GET /groups/{id}/mission`
- `POST /groups/{id}/relationship-choices`
- `GET /connections`

각 API마다 다음을 정의하라.

- 목적
- Method
- URL
- Auth 여부
- Request JSON
- Response JSON
- Error code
- 권한 조건

## 4. Group Matching Engine

이 앱의 핵심은 1:1 데이팅 매칭이 아니라 **3~5명 그룹 구성**이다.

매칭 입력 신호:

- GIVE ↔ GET 상호보완성
- 공통 관심사
- 활동 선호
- 가능한 날짜/시간
- 이동 가능한 지역/거리
- 관계 목적
- 연령 범위
- Trust / reliability
- 노쇼 기록
- 기존 만남 기록
- 같은 사람 반복 노출 제한
- 그룹 다양성
- Novelty

초기에는 머신러닝보다 설명 가능한 rule-based scoring을 우선하라.

예시 초깃값:

```text
Match Score =
0.28 * Complementarity
+ 0.22 * Interest
+ 0.18 * TimeLocation
+ 0.12 * Reliability
+ 0.10 * Diversity
+ 0.10 * Novelty
```

하지만 위 가중치는 가설일 뿐이므로 더 합리적인 구조가 있으면 수정하라.

다음을 설계하라.

1. Candidate filtering
2. Pair score 생성
3. Group score 계산
4. 3~5명 그룹 생성
5. Hard constraint
6. Soft constraint
7. 동률 처리
8. 부족한 인원 처리
9. 매칭 실패 fallback
10. Cold Start 대응

Python 또는 pseudocode를 포함하라.

## 5. AI Mission Engine

AI는 사람을 평가하는 역할이 아니라 모임 진행자 역할을 한다.

입력 데이터:

- 참가자 수
- 각 참가자의 GIVE
- 각 참가자의 GET
- INTEREST
- 활동 카테고리
- 모임 총 시간
- 장소 유형
- 안전 제한

출력은 자유 문장이 아니라 JSON Schema 기반 구조화 데이터로 제한하라.

예:

```json
{
  "title": "서로 인생사진 찍어주기",
  "goal": "각자 마음에 드는 사진 3장 얻기",
  "total_minutes": 90,
  "steps": [
    {
      "order": 1,
      "minutes": 10,
      "title": "가벼운 소개",
      "instruction": "각자 오늘 기대하는 것을 한 문장으로 말합니다."
    }
  ],
  "safety_notes": []
}
```

다음을 작성하라.

- System prompt
- User prompt template
- JSON Schema
- validation rule
- 실패 시 fallback mission
- unsafe mission 방지 rule
- hallucination 방지 방식

## 6. Relationship Choice System

활동 종료 후에만 관계 선택이 가능하도록 설계하라.

선택지 예:

- 다시 그룹 활동에서 만나고 싶음
- 다른 활동을 같이 하고 싶음
- 프로젝트/업무 연결
- 개인적으로 더 알아보고 싶음
- 선택 없음

개인적 관심은 **상호 선택일 때만** 공개해야 한다.

A → B ❤️
B → A 없음

인 경우 어떤 알림도 발생하지 않는다.

A → B ❤️
B → A ❤️

인 경우에만 mutual connection과 1:1 채팅을 생성한다.

DB 및 transaction race condition까지 고려하라.

## 7. Safety Architecture

오프라인 만남 서비스이므로 안전 기능을 MVP 핵심 기능으로 다룬다.

다음을 설계하라.

- 본인인증
- 최소 프로필 검증
- 신고
- 차단
- 노쇼 기록
- 당일 취소
- 반복 신고 사용자 처리
- 위험 사용자 탐지
- 첫 만남 장소 제한
- 개인 연락처 비노출
- 정확한 집 주소 저장 금지
- 위치정보 최소 수집
- 관리자 moderation queue
- emergency reporting flow
- 데이터 보존 정책

Trust Score를 단순 공개 숫자로 보여주지 말고 사용자 친화적 신뢰 지표로 변환하는 방식도 제안하라.

## 8. Technology Stack

MVP를 빠르게 개발한다는 기준으로 다음을 비교하고 최종 권고안을 작성하라.

### Mobile
- Flutter
- React Native

### Backend
- Supabase
- Firebase
- Node.js/NestJS
- FastAPI

### Database
- PostgreSQL
- Firestore

### Auth
- Supabase Auth
- Firebase Auth
- 국내 휴대폰 인증 provider 연동

### 기타
- Push Notification
- Realtime Chat
- Maps / Places
- AI API
- Payment
- Analytics
- Error monitoring
- Admin dashboard
- Cloud hosting

선정 이유뿐 아니라 향후 10만 MAU 이상에서의 migration risk도 설명하라.

## 9. Repository Architecture

실제 Git repository를 기준으로 폴더 구조를 설계하라.

예:

```text
/apps
  /mobile
  /admin
/services
  /api
/packages
  /shared
  /matching
  /ai-mission
/docs
  /architecture
  /api
  /product
/supabase
/tests
```

각 폴더의 책임을 설명하라.

## 10. Analytics Event 설계

MVP 단계부터 다음 funnel을 측정할 수 있어야 한다.

```text
install
→ signup_complete
→ profile_complete
→ activity_view
→ activity_join
→ group_confirmed
→ checkin
→ activity_complete
→ relationship_choice
→ mutual_connection
→ second_meeting
```

각 이벤트에 필요한 property를 정의하라.

핵심 KPI:

- Signup → First Join Conversion
- Join → Attendance Rate
- No-show Rate
- Activity Completion Rate
- Satisfaction
- Mutual Connection Rate
- 7/30 day retention
- Second Meeting Rate

## 11. Development Roadmap

다음 단계로 구분하여 개발 순서를 작성하라.

- Sprint 0: Product / Architecture
- Sprint 1: Auth / Profile
- Sprint 2: Activity
- Sprint 3: Group Matching
- Sprint 4: Offline Check-in / Mission
- Sprint 5: Relationship / Chat
- Sprint 6: Safety / Admin / Analytics
- Closed Beta

각 Sprint마다

- 목표
- 기능
- Definition of Done
- 테스트 항목
- 리스크

를 작성하라.

## 12. Critical Review

마지막에는 이 서비스의 기술 및 제품 관점에서 가장 위험한 문제 10가지를 작성하라.

특히 다음을 냉정하게 검토하라.

- 사용자가 충분하지 않아 매칭이 안 되는 문제
- 성비 불균형
- 특정 목적 사용자 유입
- 데이팅 앱으로 변질될 위험
- 노쇼
- 안전사고
- 초기 그룹 품질
- AI Mission의 실효성
- 개인정보
- 운영비 증가

각 위험마다 예방책과 관찰할 metric을 제시하라.

결과는 아이디어 제안서 수준이 아니라 **개발팀이 바로 업무를 나눌 수 있는 Software/Product Specification** 수준으로 작성하라.

---

## Project Rule

모든 기능을 설계할 때 아래 질문을 먼저 적용한다.

> 이 기능은 사용자가 사람을 소비하게 만드는가, 아니면 함께 경험하게 만드는가?

사람을 소비하는 방향이면 제거하거나 수정한다.
