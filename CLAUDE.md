# Midnight Post Office — CLAUDE.md

> 이 파일은 Claude Code가 프로젝트 세션 시작 시 가장 먼저 읽는 컨텍스트 파일이다.
> 새 세션에서도 이전 작업을 즉시 이어받을 수 있도록 최신 상태를 유지한다.

---

## 프로젝트 기본 정보

| 항목 | 내용 |
|------|------|
| 프로젝트명 | Midnight Post Office (가칭) |
| 내부 코드명 | NC 게임 프로젝트 |
| 장르 | 소셜 네트워크 / 커뮤니티 감성 게임 |
| 플랫폼 | Android, iOS, Windows (Flutter 크로스플랫폼) |
| GitHub | https://github.com/THe-L00t/wip-midnight-post-office |
| 프로젝트 경로 | `/Users/the-l00t/NC 게임 프로젝트/` |
| 기획 문서 경로 | `/Users/the-l00t/NC 게임 프로젝트/기획/` |

---

## 핵심 게임 컨셉 (반드시 숙지)

**"사람이 직접 접속해야 세계가 움직인다"**

- 플레이어는 각자 우체국을 운영하며, 편지는 **자동 전달되지 않는다**
- 편지 전달은 반드시 **사람이 직접 접속 → 관계 있는 우체국 방문 → 수동 전달**
- **관계 조건**: 두 유저 사이 편지 3번 이상 오고 가면 자동 관계 형성
- 관계 형성 후에만 상대 우체국 방문 가능 + 라우팅 경로로 활용 가능
- **중앙 우체국**: 모두가 접근 가능한 공공 공간. 관계 없는 유저에게 편지 보낼 때 진입점. **월 3회 제한**
- 인간관계가 곧 맵이 되는 구조

---

## 기술 스택

| 레이어 | 기술 |
|--------|------|
| 클라이언트 | Flutter (Dart), Riverpod, go_router, flutter_animate, Hive |
| 백엔드 | Supabase (PostgreSQL + Auth + Realtime + Storage + Edge Functions) |
| 서버 로직 | Supabase Edge Functions (Deno / TypeScript) |
| 푸시 알림 | Firebase FCM (무료, Android/iOS/Windows 통합) |
| 자동 빌드 | GitHub Actions (Windows 빌드는 Windows Runner 필요) |

> pg_cron 미사용: 자동 전달 스케줄러 불필요 (유저 액션 기반)

---

## 기획 문서 목록

| 파일 | 내용 |
|------|------|
| `기획/기획안.md` | 게임 기획 전문 (컨셉, 시스템, 감성 방향) |
| `기획/개발자_기획안.md` | DB 설계, Edge Function 코드, 서버 비용 계획 |
| `기획/AI_개발_가이드.md` | AI/사람 역할 분담, 테스트 전략, 착수 체크리스트 |

---

## 현재 진행 상태

### 완료된 것
- [x] 환경 점검 (Flutter 3.35.4, Android SDK, Xcode, VS Code 정상)
- [x] CocoaPods 재설치 완료 (v1.16.2)
- [x] 기획안 작성 완료 (기획안.md)
- [x] 개발자 기획안 작성 완료 (개발자_기획안.md)
- [x] AI 개발 가이드 작성 완료 (AI_개발_가이드.md)
- [x] GitHub 리포지토리 생성 및 연결 (wip-midnight-post-office, public)

### 미완료 — 사람이 처리해야 할 것
- [ ] **iOS Simulator 런타임 설치** (`xcodebuild -downloadPlatform iOS` 또는 Xcode → Platforms)
- [ ] **Apple Developer Program 가입** (연 $99, 승인 1~2일 소요 — 지금 바로 신청 권장)
- [ ] **Supabase 프로젝트 생성** → URL / anon key / service_role key 확보
- [ ] **Firebase 프로젝트 생성** → google-services.json / GoogleService-Info.plist 다운로드
- [ ] **기획 미결 항목 결정** (아래 참고)

### 미결 기획 항목 (개발 전 결정 필요)
- [ ] 수신자를 어떻게 찾나? (닉네임 검색 / 코드 입력 / 중앙우체국 통해서만)
- [ ] 신규 유저 첫 편지 발송 방법 (관계 0명일 때 → 중앙우체국이 유일한 진입점인지)
- [ ] 경유자가 편지 내용을 볼 수 있는지 여부
- [ ] 수익 모델 방향 (꾸미기 아이템 판매 / 중앙우체국 추가 이용권 등)
- [ ] 앱 번들 ID 확정 (예: `com.nc.midnightpostoffice`)

---

## DB 핵심 테이블 목록 (설계 완료)

| 테이블 | 역할 |
|--------|------|
| `users` | 유저 기본 정보 |
| `post_offices` | 우체국 (테마, 인테리어, decoration JSONB) |
| `letter_exchanges` | 편지 교환 횟수 추적 (관계 형성 판정용) |
| `relationships` | 관계망 (방문 가능 + 라우팅 경로, 양방향) |
| `letters` | 편지 본체 (current_holder, route[], hop_index) |
| `letter_logs` | 경유 기록 (누가 어디서 어디로 전달했는지) |
| `stamps` | 우표 컬렉션 |
| `unlocks` | 관계 해금 요소 |
| `central_post_office_letters` | 중앙 우체국 등록 편지 |
| `central_post_office_usage` | 중앙 우체국 월별 이용 횟수 |

---

## Edge Function 목록 (구현 예정)

| 함수 | 역할 |
|------|------|
| `route-letter` | 편지 발송 시 BFS 최단 경로 탐색 |
| `relay-letter` | 유저 직접 전달 처리 (권한 검증 + 상태 업데이트 + 보상) |
| `register-to-central` | 중앙 우체국 편지 등록 (월 3회 제한 포함) |
| `pickup-from-central` | 중앙 우체국 편지 픽업 |
| `send-push` | FCM 푸시 알림 발송 |

---

## 개발 로드맵

| Phase | 내용 | 상태 |
|-------|------|------|
| Phase 1 | Flutter 프로젝트 세팅, DB 스키마, 인증, 편지 기본 시스템, 중앙 우체국 | 대기 중 |
| Phase 2 | 관계 자동 형성, 관계망 시각화, 경유 보상, 우표 | 대기 중 |
| Phase 3 | 우체국 꾸미기, 해금 요소, 특별 오브젝트, Windows UI | 대기 중 |
| Phase 4 | 관리자 대시보드, 인앱 결제, CI/CD | 대기 중 |

---

## 월 예상 서버 비용

| 단계 | MAU | 비용 |
|------|-----|------|
| MVP | ~1,000 | $0 |
| 초기 | ~5,000 | $25 |
| 성장 | ~10,000 | $50~75 |

---

## AI 작업 규칙

1. **환경변수 하드코딩 금지** — 모든 키는 `.env`에서 읽기
2. **편지 전달 권한 검증은 서버(Edge Function)에서만** — 클라이언트 신뢰 금지
3. **RLS 항상 활성화** — 관계 없는 우체국 데이터 접근 차단
4. **에러 메시지는 세계관에 맞게** — "오류 발생" ❌ / "편지가 길을 잃었어요" ✅
5. **pg_cron 사용 금지** — 자동 전달 없음, 유저 액션 기반
6. **라우팅 경로(`route[]`) 계산은 서버에서만** — 클라이언트 조작 방지
7. **새 기능 추가 전 기획 문서와 정합성 확인**

---

## 환경 상태 (2026-05-19 기준)

| 도구 | 버전 | 상태 |
|------|------|------|
| Flutter | 3.35.4 | 정상 (업데이트 가능) |
| Dart | 3.9.2 | 정상 |
| Android SDK | 36.1.0 | 정상 |
| Android Studio | 2025.1 | 정상 |
| Xcode | 26.0.1 | CocoaPods 정상 / Simulator 미설치 |
| CocoaPods | 1.16.2 | 정상 |
| VS Code | 1.104.0 | 정상 |
| Node.js | 22.19.0 | 정상 |
| Git | 2.50.1 | 정상 |
