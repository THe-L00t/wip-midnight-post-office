# Midnight Post Office — AI 에이전트 개발 가이드

> AI 에이전트(Claude Code 등)가 이 프로젝트를 개발할 때 참고하는 역할 분담, 작업 방식, 테스트 기준 문서.
> 사람(사용자)이 직접 해야 하는 항목과 AI가 담당하는 항목을 명확히 구분한다.

---

# 1. 역할 분담 원칙

## AI 에이전트가 담당하는 것

- 코드 작성 (Flutter, Dart, TypeScript, SQL)
- DB 스키마 설계 및 마이그레이션 파일 생성
- Edge Function 구현
- 단위 테스트 / 통합 테스트 코드 작성
- 파일 구조 생성 및 보일러플레이트 세팅
- 버그 원인 분석 및 수정
- 리팩토링
- 문서 작성 및 갱신

## 사람(사용자)이 직접 해야 하는 것

- 외부 서비스 계정 생성 및 로그인
- 환경 변수 / 시크릿 키 발급 및 입력
- 앱 스토어 / 플레이스토어 등록
- Apple 개발자 계정, 인증서, 프로비저닝 프로파일 관리
- Firebase 프로젝트 생성 및 `google-services.json` / `GoogleService-Info.plist` 다운로드
- Supabase 프로젝트 생성 및 URL / anon key 확인
- 실제 디바이스에서 최종 확인 (AI는 시뮬레이터 기반 판단만 가능)
- 기획 방향 결정 및 최종 승인
- 인앱 결제 상품 등록 (스토어 콘솔 직접 접근 필요)
- 게임 출시 심사 제출

---

# 2. 개발 시작 전 사람이 준비해야 할 것

아래 항목들은 AI가 코드를 작성하기 전에 사람이 먼저 완료해야 한다.
완료 후 관련 값을 AI에게 전달하면 코드에 반영한다.

## 2-1. Supabase 설정

```
[ ] Supabase 계정 생성 (supabase.com)
[ ] 새 프로젝트 생성
[ ] 프로젝트 URL 확인         → SUPABASE_URL
[ ] anon public key 확인     → SUPABASE_ANON_KEY
[ ] service_role key 확인    → SUPABASE_SERVICE_ROLE_KEY (Edge Function용, 절대 클라이언트에 노출 금지)
```

## 2-2. Firebase 설정

```
[ ] Firebase 프로젝트 생성 (console.firebase.google.com)
[ ] Android 앱 등록 → google-services.json 다운로드 → android/app/ 에 배치
[ ] iOS 앱 등록     → GoogleService-Info.plist 다운로드 → ios/Runner/ 에 배치
[ ] Windows 앱 등록 → firebase_options.dart 생성 (flutterfire configure 실행)
[ ] FCM 서버 키 확인 → Edge Function 환경변수로 등록
```

## 2-3. Apple 개발자 설정 (iOS 빌드용)

```
[ ] Apple Developer Program 가입 (연 $99)
[ ] Xcode에서 Signing & Capabilities 설정
[ ] Bundle ID 확정
[ ] CocoaPods 정상 동작 확인 (flutter doctor 기준)
```

## 2-4. 환경변수 파일 준비

AI가 생성하는 `.env.example`을 참고하여 실제 값을 채운 `.env` 파일을 직접 작성한다.
`.env` 파일은 절대 Git에 커밋하지 않는다.

```
SUPABASE_URL=https://xxxxxxxxxxx.supabase.co
SUPABASE_ANON_KEY=eyJ...
SUPABASE_SERVICE_ROLE_KEY=eyJ...
FCM_SERVER_KEY=AAAA...
```

---

# 3. AI 에이전트 작업 순서 (Phase별)

## Phase 1 — 프로젝트 초기 세팅

AI가 순서대로 수행:

1. Flutter 프로젝트 생성
   ```
   flutter create midnight_post_office --org com.nc.mpo
   ```

2. 디렉토리 구조 생성 (features, core, shared 분리)

3. `pubspec.yaml` 의존성 추가

4. Supabase 클라이언트 초기화 코드 작성
   - `.env` 값을 읽어 주입하는 방식으로 구성
   - 하드코딩 금지

5. DB 마이그레이션 파일 작성 (`supabase/migrations/`)
   - users, post_offices, letter_exchanges, relationships
   - letters, letter_logs, stamps, unlocks
   - central_post_office_letters, central_post_office_usage

6. RLS 정책 파일 작성

7. Edge Function 파일 생성
   - route-letter
   - relay-letter
   - register-to-central
   - pickup-from-central
   - send-push

> **사람이 할 일**: 마이그레이션 파일 생성 후 Supabase 대시보드 또는 CLI로 직접 실행
> ```
> ! supabase db push
> ```

---

## Phase 2 — 인증 / 우체국 기본 UI

AI가 수행:
- Google OAuth 로그인 화면 구현
- 회원가입 시 `users` + `post_offices` 레코드 자동 생성 로직
- 내 우체국 화면 기본 구조

> **사람이 할 일**:
> - Supabase Auth에서 Google provider 활성화 (대시보드 직접 설정)
> - OAuth 클라이언트 ID를 Google Cloud Console에서 발급
> - iOS용 URL Scheme 설정 (Xcode에서 직접)

---

## Phase 3 — 편지 시스템

AI가 수행:
- 편지 작성 화면
- BFS 경로 탐색 Edge Function (`route-letter`)
- 전달 액션 Edge Function (`relay-letter`)
- 편지함(대기 편지 목록) 화면
- Supabase Realtime 구독 코드

---

## Phase 4 — 중앙 우체국

AI가 수행:
- 중앙 우체국 화면
- 편지 등록 / 픽업 Edge Function
- 월 이용 횟수 표시 UI

---

## Phase 5 — 관계망 / 감성 콘텐츠

AI가 수행:
- 관계 자동 형성 판정 SQL 함수
- 관계망 시각화 화면
- 우체국 꾸미기 시스템
- 해금 요소 로직

---

# 4. AI에게 작업을 요청하는 방법

## 효과적인 요청 예시

```
# 좋은 요청
"relay-letter Edge Function을 작성해줘.
 현재 편지 보유자 검증, 다음 홉 계산, 경유 기록 저장,
 보상 지급, FCM 알림 발송까지 포함해서."

"편지함 화면을 만들어줘.
 대기 중인 편지 목록을 보여주고,
 각 편지를 탭하면 상세 내용과 전달하기 버튼이 나와야 해."
```

```
# 피해야 할 요청
"게임 만들어줘."         ← 범위가 너무 넓음
"예쁘게 만들어줘."       ← 기준 없음, 구체적인 디자인 방향 필요
"알아서 해줘."           ← 판단 기준 없음
```

## 요청할 때 함께 주면 좋은 것

- 어떤 화면인지 (화면 이름, 진입 경로)
- 어떤 데이터를 보여줘야 하는지
- 어떤 액션이 있어야 하는지 (버튼, 탭, 스와이프 등)
- 플랫폼 분기가 필요한지 (모바일/데스크탑 다르게)
- 에러 케이스를 어떻게 처리할지

---

# 5. 테스트 전략

## 5-1. AI가 작성하는 테스트

### 단위 테스트 (Unit Test)

순수 로직만 테스트. 외부 의존성 없음.

```dart
// test/unit/letter_routing_test.dart
// BFS 경로 탐색 알고리즘이 올바른 경로를 반환하는지 검증
void main() {
  test('직접 연결된 유저에게 편지 경로 탐색', () {
    // ...
  });

  test('중간 경유자 1명을 통한 경로 탐색', () {
    // ...
  });

  test('경로 없을 때 null 반환', () {
    // ...
  });

  test('순환 방지: 같은 노드 두 번 방문 안 함', () {
    // ...
  });
}
```

```dart
// test/unit/exchange_count_test.dart
// 교환 횟수 3회 이상 시 관계 형성 판정 로직
void main() {
  test('교환 횟수 2회 → 관계 미형성', () { ... });
  test('교환 횟수 3회 → 관계 형성', () { ... });
  test('user_a < user_b 정렬 저장 검증', () { ... });
}
```

```dart
// test/unit/central_usage_test.dart
// 월 이용 횟수 제한 로직
void main() {
  test('이번 달 첫 사용 → 성공', () { ... });
  test('3회 사용 후 추가 시도 → 실패(429)', () { ... });
  test('다음 달이 되면 횟수 리셋', () { ... });
}
```

---

### 위젯 테스트 (Widget Test)

UI 컴포넌트 단위 테스트.

```dart
// test/widget/letter_box_test.dart
void main() {
  testWidgets('대기 중인 편지가 없을 때 빈 상태 메시지 표시', (tester) async {
    // ...
  });

  testWidgets('편지 목록이 있을 때 카드 렌더링', (tester) async {
    // ...
  });

  testWidgets('전달하기 버튼 탭 → relay-letter 호출', (tester) async {
    // ...
  });
}
```

---

### Edge Function 테스트 (Deno)

```typescript
// supabase/functions/relay-letter/relay-letter.test.ts
import { assertEquals } from "https://deno.land/std/assert/mod.ts";

Deno.test("현재 보유자가 아닌 유저가 전달 시도 → 403 반환", async () => {
  // ...
});

Deno.test("마지막 홉 전달 시 status가 delivered로 변경", async () => {
  // ...
});

Deno.test("전달 후 letter_logs에 기록 생성", async () => {
  // ...
});
```

---

## 5-2. 사람이 직접 수행하는 테스트

AI는 코드를 작성하고 시뮬레이터 기준 판단은 가능하지만,
아래 항목은 **실제 디바이스**에서 사람이 직접 확인해야 한다.

### 실기기 필수 확인 항목

```
[ ] Android 실기기 — 푸시 알림 수신 확인 (FCM)
[ ] iOS 실기기    — 푸시 알림 수신 확인 (APNs + FCM)
[ ] Windows PC   — 데스크탑 레이아웃 렌더링 확인
[ ] iOS 실기기    — Apple 로그인 동작 확인
[ ] Android      — 뒤로가기 버튼 동작 확인
[ ] 저사양 Android — 애니메이션 성능 확인
```

### 다계정 시나리오 테스트 (사람 2명 이상 필요)

```
[ ] 유저 A, B, C 계정 3개로 편지 전달 전 과정 실행
    A → 중앙우체국 → B 픽업 → B 방문전달(C) → C 수신

[ ] 편지 교환 3회 후 관계 형성 확인
    (A↔B 편지 3번 주고받음 → 관계 자동 생성 → 우체국 방문 가능 여부 확인)

[ ] 중앙 우체국 월 3회 제한 확인
    (3회 사용 후 4번째 시도 → 제한 안내 메시지 표시 확인)

[ ] 관계 없는 유저의 우체국 직접 접근 시도 → 차단 확인
```

---

## 5-3. DB 정합성 테스트

Supabase SQL Editor에서 직접 실행하여 확인.
AI가 쿼리를 작성하고, 사람이 콘솔에서 실행하는 방식으로 협업.

```sql
-- 관계 형성 후 양방향 행이 모두 존재하는지 확인
SELECT * FROM relationships
WHERE (from_user_id = 'A' AND to_user_id = 'B')
   OR (from_user_id = 'B' AND to_user_id = 'A');
-- 결과: 2행 이어야 함

-- 편지 경로 배열이 올바르게 저장되었는지
SELECT id, route, hop_index, current_holder, status
FROM letters
WHERE sender_id = 'A'
ORDER BY created_at DESC
LIMIT 5;

-- 중앙 우체국 이번 달 이용 횟수
SELECT * FROM central_post_office_usage
WHERE user_id = 'A'
  AND year_month = TO_CHAR(now(), 'YYYY-MM');
```

---

## 5-4. 테스트 실행 명령어

```bash
# 단위 + 위젯 테스트 전체 실행
flutter test

# 특정 파일만
flutter test test/unit/letter_routing_test.dart

# Edge Function 테스트 (Deno)
deno test supabase/functions/relay-letter/relay-letter.test.ts

# 커버리지 확인
flutter test --coverage
genhtml coverage/lcov.info -o coverage/html
```

---

# 6. AI가 코드 작성 시 반드시 지키는 규칙

## 보안

- 환경변수는 반드시 `.env`에서 읽어야 함. 코드에 직접 하드코딩 절대 금지.
- `SUPABASE_SERVICE_ROLE_KEY`는 Edge Function 서버 사이드에서만 사용. 클라이언트 코드에 포함 금지.
- 편지 전달 권한 검증은 클라이언트가 아닌 **Edge Function 서버에서** 수행.
- RLS는 모든 테이블에 활성화 상태 유지.

## 코드 품질

- Flutter: Riverpod으로 상태 관리, `ref.watch` / `ref.read` 용도 구분 준수
- SQL: 모든 쿼리에 인덱스 활용 여부 검토 후 작성
- Edge Function: 모든 에러 케이스에 명시적 HTTP 상태 코드 반환
- 한 함수가 하나의 역할만 수행 (경유 보상 / 관계 판정 / 알림 발송 분리)

## 감성 / UX

- 에러 메시지도 게임 세계관에 맞게 작성
  - "오류 발생" ❌
  - "편지가 길을 잃었어요. 잠시 후 다시 시도해주세요." ✅
- 로딩 상태에도 감성 연출 (빈 화면 대신 우편함 애니메이션 등)
- 관계 없는 우체국 접근 차단 시 "아직 이 우체국과 연결되지 않았어요" 형태로 안내

---

# 7. 자주 발생할 수 있는 문제와 대응

| 상황 | AI 대응 | 사람이 할 일 |
|------|---------|------------|
| BFS 경로 탐색 결과 없음 | "아직 닿는 길이 없어요" UI 표시 코드 작성 | 중앙 우체국 이용 유도 기획 확인 |
| FCM 토큰 갱신 실패 | 토큰 갱신 재시도 로직 작성 | Firebase 콘솔에서 앱 등록 상태 확인 |
| Supabase Realtime 연결 끊김 | 자동 재연결 로직 작성 | Supabase 대시보드 Realtime 설정 확인 |
| iOS 빌드 실패 (CocoaPods) | 오류 메시지 분석 후 해결책 제시 | 터미널에서 직접 `pod install` 실행 |
| RLS로 인한 데이터 조회 실패 | RLS 정책 조건 재검토 및 수정 | Supabase 대시보드에서 정책 적용 확인 |
| 중앙 우체국 편지 중복 픽업 | `picked_up_by IS NULL` 조건 + 트랜잭션으로 처리 | DB에서 직접 데이터 확인 |
| 월 이용 횟수 오차 | upsert 트랜잭션 검토 | Supabase SQL Editor로 수동 확인 |

---

# 8. Git 브랜치 전략

```
main        ← 출시 가능한 안정 버전만
develop     ← 통합 개발 브랜치
feature/*   ← 기능 단위 브랜치 (AI가 작업 단위로 생성)
fix/*       ← 버그 수정
```

AI는 작업 시 `feature/` 브랜치에서 작업하고,
사람이 `develop`으로 머지 여부를 최종 확인한다.

---

# 9. 체크리스트 — Phase 1 착수 전 최종 확인

```
사람이 완료해야 할 것:
[ ] Supabase 프로젝트 생성 완료
[ ] Supabase URL / anon key / service_role key 확보
[ ] Firebase 프로젝트 생성 완료
[ ] google-services.json 다운로드 완료
[ ] GoogleService-Info.plist 다운로드 완료
[ ] flutter doctor → 전 항목 정상 (또는 iOS Simulator 이슈 해결)
[ ] .env 파일 작성 완료

AI에게 전달하면 되는 것:
[ ] 위 .env 값들 (시크릿 키는 대화 외 안전한 방법으로 전달)
[ ] 앱 번들 ID (예: com.nc.mpo)
[ ] 원하는 앱 이름 / 아이콘 방향
[ ] Phase 1 시작 요청
```

---

*작성일: 2026-05-19*
*대상 독자: AI 에이전트 (Claude Code) + 프로젝트 담당자*
