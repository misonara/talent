# Talent MVP Product Specification v0.1

## 1. Product Definition

**Category:** Purposeful Social App

**Core promise:** 사람을 찾지 말고, 같이 할 일을 찾으세요.

Talent는 사용자의 GIVE(나눌 수 있는 것), GET(배우고 싶은 것), INTEREST(함께하고 싶은 활동)를 기반으로 3~5명의 소규모 오프라인 활동을 만들고, 활동 이후에만 관계 선택을 열어주는 목적형 관계 플랫폼이다.

### Product Rule

새 기능을 검토할 때 항상 먼저 묻는다.

> 이 기능이 사람을 소비하게 만드는가, 함께 경험하게 만드는가?

사람 소비형 UX(무한 스와이프, 인기순 사람 랭킹, 외모 중심 탐색)는 MVP에서 제외한다.

---

## 2. Initial Target

- 한국 27~39세 직장인
- 직장 밖에서 새로운 사람을 만나기 어려운 사용자
- 소개팅 앱의 외모/스와이프 구조에 피로감을 느끼는 사용자
- 동호회 가입은 부담스럽지만 새로운 활동과 사람을 원하는 사용자
- 연애가 최우선 목적은 아니지만 좋은 인연에는 열려 있는 사용자

### Initial activity categories

Closed Beta에서는 3개로 제한한다.

1. 영어
2. 사진
3. 산책/러닝

검증 이후 커피/맛집, 자기계발/사이드 프로젝트를 확장한다.

---

## 3. Core Hypothesis

**H1.** 명확한 활동 목적이 있으면 낯선 사람끼리의 첫 만남 부담이 낮아진다.

**H2.** 3~5명의 소규모 그룹은 1:1보다 부담이 적고 대형 모임보다 실제 대화가 잘 발생한다.

**H3.** GIVE ↔ GET 상호보완성이 단순 공통 관심사보다 관계 형성에 유용하다.

**H4.** 활동 이후 상호 선택 방식은 거절 부담을 줄이고 안전한 관계 확장을 돕는다.

**H5.** 사용자가 좋은 경험을 하면 같은 참가자 또는 같은 유형의 활동에 다시 참여한다.

### North Star Metric

**Second Meeting Rate**

한 번 함께 활동한 사용자가 30일 이내 다시 같은 사람과 활동하거나, 앱을 통해 후속 활동을 만드는 비율.

---

## 4. MVP Feature Freeze

MVP 기능은 아래 15개로 고정한다.

### 1. Account & Phone Verification
- 휴대폰 기반 가입/로그인
- 최소 연령 확인
- 이용약관/개인정보/안전정책 동의

### 2. Core Profile
- 닉네임
- 프로필 사진 1~3장
- 연령대
- 직업/분야(선택)
- 한 줄 소개
- 본인인증 상태

프로필은 얼굴보다 활동 역량과 관심사가 우선 노출된다.

### 3. GIVE / GET / INTEREST Setup
- GIVE 최대 5개
- GET 최대 5개
- INTEREST 최대 10개
- 숙련도는 자기평가 수준으로만 사용하고 랭킹에는 사용하지 않는다.

### 4. Availability & Activity Area
- 가능한 요일/시간대
- 활동 가능 지역
- 세부 실시간 위치는 저장하지 않는다.

### 5. Activity Discovery
홈 화면은 사람이 아니라 활동 카드 중심이다.

카드 필수 정보:
- 활동 제목
- 카테고리
- 일정
- 지역
- 예상 시간
- 모집 인원
- 활동 목표

### 6. Join / Cancel Activity
- 참여 신청
- 취소
- 마감 상태
- 대기 상태

MVP에서는 사용자의 자유로운 공개 모임 생성 기능은 제외하고 운영진/시스템이 활동 템플릿을 공급한다.

### 7. Group Formation
- 목표 4명
- 허용 범위 3~5명
- 시간/지역/관심사/GIVE↔GET/신뢰 신호를 기준으로 그룹 구성
- 인원 부족 시 fallback 처리

### 8. Group Chat
- 그룹 확정 후 채팅 활성화
- 1:1 DM은 활동 종료 전 비활성화
- 전화번호/개인 연락처 공개를 기본적으로 유도하지 않는다.

### 9. Check-in
- 활동 시작 전후 제한된 시간창에서 체크인
- 출석/노쇼 측정
- 정확한 GPS 상시 추적은 하지 않는다.

### 10. AI Mission
- 그룹 구성과 활동 카테고리에 맞춘 진행 순서 제공
- AI는 사람의 매력/등급/연애 가능성을 평가하지 않는다.
- 안전한 템플릿 범위 안에서만 활동을 생성한다.

### 11. Activity Completion & Review
- 활동 완료 처리
- 만족도
- 다시 이런 활동을 하고 싶은지
- 안전/불편 피드백
- 다른 참가자에 대한 공개 별점 랭킹은 제공하지 않는다.

### 12. Relationship Choice
활동 종료 후에만 참가자별로 선택 가능:
- 다시 그룹 활동에서 만나고 싶음
- 다른 활동을 같이 하고 싶음
- 프로젝트/업무로 연결하고 싶음
- 개인적으로 더 알아보고 싶음
- 선택 없음

상대방에게 단방향 선택 사실을 노출하지 않는다.

### 13. Mutual Connection & 1:1 Chat
- 상호 선택일 때만 Connection 생성
- 이후 1:1 Chat 활성화
- connection 해제/차단 가능

### 14. Safety: Report / Block / Trust Indicators
- 신고
- 차단
- 노쇼 기록
- 반복 취소 기록
- 본인인증 여부
- 활동 참여 횟수
- 약속 이행 상태

내부 Trust Score 숫자는 사용자에게 직접 공개하지 않는다.

### 15. Minimal Admin Moderation
운영자용 최소 기능:
- 사용자 검색
- 신고 확인
- 계정 제한/정지
- 활동 관리
- 노쇼/반복 신고 확인

---

## 5. Explicitly Out of MVP

다음 기능은 핵심 가설 검증 전에는 만들지 않는다.

### Phase 2
- 사용자 직접 모임 생성
- 세부 AI 개인 추천 고도화
- Venue 자동 추천/예약
- 활동 보증금
- 간편결제
- 고급 신뢰 프로필

### Phase 3
- PLUS 구독
- Premium Matching
- Host Marketplace
- Venue Partnership 수익화
- 대학/기업 Community 프로그램
- 브랜드 협찬

### Never / Avoid by default
- 외모 기반 무한 스와이프
- 인기 사용자 랭킹
- 공개 좋아요 수
- 공개 팔로워 경쟁
- 모임 전 무제한 1:1 DM

---

## 6. Core User Journey

1. 앱 설치
2. 휴대폰 인증
3. Core Profile
4. GIVE 설정
5. GET 설정
6. INTEREST 설정
7. 가능 시간/활동 지역 설정
8. Home에서 활동 탐색
9. 활동 상세 확인
10. 참가 신청
11. 3~5명 그룹 확정
12. 그룹 채팅 및 사전 안내
13. 공개된 장소 방문
14. Check-in
15. AI Mission 진행
16. 활동 완료
17. 후기 및 안전 피드백
18. 관계 선택
19. 상호 선택이면 Mutual Connection
20. 1:1 Chat 또는 다음 활동

---

## 7. Key UX Constraints

1. Home 첫 화면에 사람 카드 목록을 배치하지 않는다.
2. 활동 신청까지 최대한 적은 단계로 만든다.
3. 가입 시 프로필을 완벽하게 작성하도록 강요하지 않는다.
4. 상대의 단방향 호감 선택은 절대 공개하지 않는다.
5. 활동 전 개인 DM은 제한한다.
6. 첫 모임은 공개된 장소 중심으로 운영한다.
7. 정확한 주거지나 상시 GPS 위치는 수집하지 않는다.
8. 사용자 평가를 공개 인기 점수로 만들지 않는다.
9. 노쇼는 제품 품질 핵심 문제로 취급한다.
10. AI가 사람의 가치/매력/연애 가능성을 점수화하지 않는다.

---

## 8. MVP Success Metrics

### Acquisition / Activation
- Signup Completion Rate
- Profile Completion Rate
- Signup → First Activity Join Conversion

### Offline Quality
- Group Confirmation Rate
- Join → Check-in Rate
- No-show Rate
- Activity Completion Rate
- Activity Satisfaction

### Relationship
- Relationship Choice Rate
- Mutual Connection Rate
- Second Meeting Rate

### Retention
- 7-day Retention
- 30-day Retention
- 30-day Second Activity Participation

### Initial validation targets
정확한 목표값은 Closed Beta 시작 전 기준선을 정한다. 초기에는 절대 수치보다 퍼널 병목과 사용자 인터뷰를 함께 본다.

---

## 9. Closed Beta Scope

- 사용자: 50~100명
- 지역: 한 개 생활권
- 카테고리: 영어 / 사진 / 산책·러닝
- 그룹: 3~5명, 기본 4명
- 활동: 운영진/시스템 생성 템플릿 중심

Closed Beta에서 우선 검증할 질문:

1. 실제로 모임이 성사되는가?
2. 신청자가 실제로 참석하는가?
3. 첫 만남의 어색함이 줄어드는가?
4. AI Mission이 실제로 도움이 되는가?
5. 다시 만나고 싶은 사람이 생기는가?
6. 두 번째 활동이 발생하는가?

---

## 10. Product Risks

### Cold Start
지역별 사용자 밀도가 낮으면 그룹이 형성되지 않는다.

**대응:** 초기 지역/시간/카테고리를 강하게 제한한다.

### Dating App Drift
사용자가 활동보다 이성 탐색을 목적으로 사람을 소비할 수 있다.

**대응:** 사람 검색/스와이프/모임 전 DM 제한, 활동 중심 홈 유지.

### Gender Imbalance
특정 성별 비율이 극단적으로 쏠릴 수 있다.

**대응:** 성비를 서비스 약속으로 강제하기보다 그룹 품질/안전/목적 일치 중심으로 운영하고, 실제 데이터로 정책을 조정한다.

### No-show
한 명의 노쇼가 4인 그룹 경험 전체를 망칠 수 있다.

**대응:** 체크인, 반복 노쇼 신호, 리마인드, 추후 보증금 검토.

### Safety Incident
오프라인 만남의 신뢰 문제가 가장 큰 위험이다.

**대응:** 본인인증, 공개 장소, 신고/차단, 운영자 moderation을 MVP에 포함한다.

### Weak Activity Quality
활동 자체가 재미없으면 관계도 생기지 않는다.

**대응:** 초기에 활동 종류를 제한하고 운영진이 품질을 직접 관리한다.

---

## 11. Decision Log v0.1

- 기본 그룹 크기: 4명
- 허용 그룹 크기: 3~5명
- Closed Beta 카테고리: 영어, 사진, 산책/러닝
- 사용자 공개 모임 생성: MVP 제외
- 결제/보증금: MVP 제외
- 구독: MVP 제외
- 모임 전 1:1 DM: 제외
- 활동 후 Mutual Connection: MVP 포함
- AI Mission: MVP 포함
- 신고/차단/운영자 moderation: MVP 포함
