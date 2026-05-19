# Apple Developer 설정 가이드

> iOS 빌드 및 App Store 배포에 필요합니다.
> 가입 심사에 1~2일 소요되므로 가장 먼저 처리하는 것을 권장합니다.

---

## Step 1. Apple Developer Program 가입

1. https://developer.apple.com/programs/ 접속
2. **Enroll** 클릭
3. Apple ID로 로그인 (없으면 먼저 생성)
4. 개인(Individual) 또는 조직(Organization) 선택
   - 개인: 연 $99, 즉시 처리 가능
   - 조직: D-U-N-S 번호 필요, 심사 1~2주 소요
5. 결제 완료 → 승인 이메일 대기 (보통 24~48시간)

- [ ] Apple Developer Program 가입 신청 완료
- [ ] 승인 이메일 수신 완료

---

## Step 2. Bundle ID 등록

1. https://developer.apple.com/account/ 접속
2. **Certificates, Identifiers & Profiles → Identifiers → +**
3. **App IDs** 선택 → **App** 선택
4. 아래 정보 입력:

| 항목 | 값 |
|------|-----|
| Description | Midnight Post Office |
| Bundle ID (Explicit) | `com.nc.midnightpostoffice` |

5. Capabilities에서 아래 항목 활성화:
   - **Push Notifications** ← FCM 필수
   - **Sign In with Apple** ← Apple 로그인 필수

6. **Register** 클릭

- [ ] Bundle ID 등록 완료
- [ ] Push Notifications 활성화 완료
- [ ] Sign In with Apple 활성화 완료

---

## Step 3. Xcode Signing 설정

> Flutter 프로젝트 생성 후 진행합니다.

1. `[프로젝트]/ios/Runner.xcworkspace` Xcode로 열기
2. 좌측 Navigator에서 **Runner** 클릭
3. **Signing & Capabilities 탭**
4. **Automatically manage signing** 체크
5. **Team**: 본인 Apple Developer 계정 선택
6. **Bundle Identifier**: `com.nc.midnightpostoffice` 입력
7. Xcode가 자동으로 Provisioning Profile 생성

- [ ] Xcode Signing 설정 완료

---

## Step 4. Sign In with Apple 설정

1. Xcode → Runner → **Signing & Capabilities**
2. **+ Capability** 클릭 → **Sign In with Apple** 추가

- [ ] Xcode Sign In with Apple Capability 추가 완료

---

## Step 5. APNs 키 생성

> Firebase APNs 등록에도 사용됩니다 (`02_Firebase_설정.md` Step 4 참고)

1. https://developer.apple.com/account/ → **Keys → +**
2. Key Name: `MPO Push Key`
3. **Apple Push Notifications service (APNs)** 체크
4. **Continue → Register**
5. **Download** 클릭 → `.p8` 파일 저장
   > 이 파일은 **딱 한 번만 다운로드 가능**합니다. 안전한 곳에 보관하세요.
6. Key ID 메모해두기 (화면에 표시됨)

- [ ] APNs 키 생성 완료
- [ ] .p8 파일 안전한 곳에 저장 완료
- [ ] Key ID 메모 완료

---

## Step 6. Team ID 확인

Firebase, Supabase Apple Provider 설정에 필요합니다.

1. https://developer.apple.com/account/ 접속
2. 우측 상단 또는 **Membership** 탭에서 **Team ID** 확인
3. 형식: `XXXXXXXXXX` (10자리 영문+숫자)

- [ ] Team ID 확인 및 메모 완료

---

## 완료 체크

- [ ] Step 1 — Apple Developer Program 가입 및 승인
- [ ] Step 2 — Bundle ID 등록 (Push + Sign In with Apple 포함)
- [ ] Step 3 — Xcode Signing 설정
- [ ] Step 4 — Sign In with Apple Capability 추가
- [ ] Step 5 — APNs 키 생성 및 보관
- [ ] Step 6 — Team ID 확인

---

## 메모 공간 (작업하면서 값 기록용)

```
Bundle ID     : com.nc.midnightpostoffice
Team ID       : (여기에 작성)
APNs Key ID   : (여기에 작성)
.p8 파일 위치 : (여기에 작성)
```
