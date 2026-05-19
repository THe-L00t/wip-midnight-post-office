# Supabase 설정 가이드

> 이 파일의 내용을 순서대로 따라하면 됩니다.
> 완료한 항목은 [ ] → [x] 로 바꿔두세요.

---

## Step 1. 프로젝트 생성

1. https://supabase.com 접속 → 로그인 (GitHub 계정 권장)
2. **New Project** 클릭
3. 아래 값으로 설정:

| 항목 | 권장값 |
|------|--------|
| Project name | `midnight-post-office` |
| Database password | 강력한 비밀번호 설정 후 **어딘가 안전하게 저장** |
| Region | Northeast Asia (Seoul) — 가장 가까운 서버 |

4. **Create new project** 클릭 → 약 1~2분 대기

---

## Step 2. 키 확인 및 기록

프로젝트 생성 완료 후:

**Settings → API** 메뉴로 이동

아래 3가지 값을 복사해서 `.env` 파일에 붙여넣기:

```
# .env 파일에 작성할 내용 (이 파일은 절대 GitHub에 올리지 마세요)

SUPABASE_URL=https://여기에_프로젝트_URL_붙여넣기.supabase.co
SUPABASE_ANON_KEY=여기에_anon_public_키_붙여넣기
SUPABASE_SERVICE_ROLE_KEY=여기에_service_role_키_붙여넣기
```

> service_role 키는 서버(Edge Function)에서만 사용합니다.
> 절대 앱 코드나 GitHub에 포함시키지 마세요.

- [ ] SUPABASE_URL 확인 완료
- [ ] SUPABASE_ANON_KEY 확인 완료
- [ ] SUPABASE_SERVICE_ROLE_KEY 확인 완료

---

## Step 3. Auth 설정 — Google 로그인

**Authentication → Providers → Google** 활성화

1. https://console.cloud.google.com 접속
2. 새 프로젝트 생성 또는 기존 프로젝트 선택
3. **API 및 서비스 → 사용자 인증 정보 → OAuth 2.0 클라이언트 ID** 생성
4. 애플리케이션 유형: **웹 애플리케이션**
5. 승인된 리디렉션 URI에 Supabase 콜백 URL 추가:
   ```
   https://[YOUR_PROJECT_ID].supabase.co/auth/v1/callback
   ```
6. 생성된 **클라이언트 ID**와 **클라이언트 보안 비밀**을 Supabase Google Provider에 입력

- [ ] Google OAuth 클라이언트 생성 완료
- [ ] Supabase Google Provider 활성화 완료

---

## Step 4. Auth 설정 — Apple 로그인 (iOS 필수)

**Authentication → Providers → Apple** 활성화

Apple Developer 계정 필요 (→ `03_Apple_Developer_설정.md` 참고)

1. Apple Developer → **Certificates, IDs & Profiles → Keys** → 새 키 생성
2. **Sign In with Apple** 활성화
3. Key ID와 .p8 파일 다운로드
4. Supabase Apple Provider에 입력:
   - Service ID
   - Team ID
   - Key ID
   - Private Key (.p8 파일 내용)

- [ ] Apple Sign In 키 생성 완료
- [ ] Supabase Apple Provider 활성화 완료

---

## Step 5. Storage 버킷 생성

**Storage → New bucket**

아래 버킷 3개 생성:

| 버킷 이름 | Public 여부 | 용도 |
|-----------|------------|------|
| `post-office-assets` | Public | 우체국 배경, 테마 이미지 |
| `stamp-assets` | Public | 우표 이미지 |
| `user-uploads` | Private | 편지 첨부파일, 프로필 이미지 |

- [ ] `post-office-assets` 버킷 생성 완료
- [ ] `stamp-assets` 버킷 생성 완료
- [ ] `user-uploads` 버킷 생성 완료

---

## Step 6. Realtime 활성화

편지 도착 실시간 알림에 사용됩니다.

**Database → Replication** 메뉴로 이동

아래 테이블에 Realtime 활성화:

| 테이블 | 활성화 이유 |
|--------|------------|
| `letters` | 편지 대기 상태 변경 실시간 감지 |
| `unlocks` | 관계 해금 즉시 알림 |

각 테이블 옆 토글 **ON** 으로 변경.

- [ ] `letters` Realtime 활성화 완료
- [ ] `unlocks` Realtime 활성화 완료

---

## Step 7. 이메일 템플릿 설정 (선택)

**Authentication → Email Templates**

기본 이메일 템플릿을 게임 세계관에 맞게 수정할 수 있습니다.

| 템플릿 | 기본 내용 | 권장 수정 방향 |
|--------|----------|--------------|
| Confirm signup | "Confirm your email" | "우체국 등록을 완료해주세요" |
| Magic Link | "Your magic link" | "당신의 우체국 열쇠가 도착했어요" |
| Reset Password | "Reset your password" | "우체국 열쇠를 재발급해드려요" |

- [ ] 이메일 템플릿 수정 완료 (선택)

---

## Step 8. DB 마이그레이션 실행

> AI(Claude Code)가 마이그레이션 파일을 작성해두면, 아래 명령어로 실행합니다.

Supabase CLI 설치 (아직 없다면):
```bash
brew install supabase/tap/supabase
```

프로젝트 연결 및 마이그레이션 실행:
```bash
cd "/Users/the-l00t/NC 게임 프로젝트"
supabase login
supabase link --project-ref [YOUR_PROJECT_ID]
supabase db push
```

- [ ] Supabase CLI 설치 완료
- [ ] 프로젝트 연결 완료
- [ ] 마이그레이션 실행 완료

---

## Step 9. Edge Functions 배포

> AI가 함수 코드를 작성한 후 아래 명령어로 배포합니다.

```bash
# 환경변수 등록
supabase secrets set FCM_SERVER_KEY=여기에_FCM_서버키

# 함수 배포
supabase functions deploy route-letter
supabase functions deploy relay-letter
supabase functions deploy register-to-central
supabase functions deploy pickup-from-central
supabase functions deploy send-push
```

- [ ] FCM_SERVER_KEY 환경변수 등록 완료
- [ ] 모든 Edge Function 배포 완료

---

## 완료 체크

- [ ] Step 1 — 프로젝트 생성
- [ ] Step 2 — 키 확인 및 .env 작성
- [ ] Step 3 — Google 로그인 설정
- [ ] Step 4 — Apple 로그인 설정
- [ ] Step 5 — Storage 버킷 생성
- [ ] Step 6 — Realtime 활성화
- [ ] Step 7 — 이메일 템플릿 설정 (선택)
- [ ] Step 8 — DB 마이그레이션 실행
- [ ] Step 9 — Edge Functions 배포
