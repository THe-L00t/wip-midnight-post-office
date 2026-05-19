# Firebase 설정 가이드

> FCM 푸시 알림 연동을 위한 Firebase 설정입니다.
> Android / iOS / Windows 3개 플랫폼 모두 등록해야 합니다.

---

## Step 1. Firebase 프로젝트 생성

1. https://console.firebase.google.com 접속 → Google 계정 로그인
2. **프로젝트 추가** 클릭
3. 프로젝트 이름: `midnight-post-office`
4. Google Analytics: 선택 사항 (나중에 켜도 됨)
5. **프로젝트 만들기** 클릭

- [ ] Firebase 프로젝트 생성 완료

---

## Step 2. Android 앱 등록

1. Firebase 콘솔 홈 → **Android 아이콘** 클릭
2. Android 패키지 이름 입력:
   ```
   com.nc.midnightpostoffice
   ```
   > 앱 닉네임: `Midnight Post Office Android`
3. **앱 등록** 클릭
4. `google-services.json` 다운로드
5. 파일 위치: Flutter 프로젝트 생성 후 아래 경로에 배치
   ```
   [프로젝트]/android/app/google-services.json
   ```

- [ ] Android 앱 등록 완료
- [ ] google-services.json 다운로드 완료
- [ ] google-services.json 올바른 경로에 배치 완료

---

## Step 3. iOS 앱 등록

1. Firebase 콘솔 홈 → **iOS 아이콘** 클릭
2. Apple 번들 ID 입력:
   ```
   com.nc.midnightpostoffice
   ```
   > 앱 닉네임: `Midnight Post Office iOS`
3. **앱 등록** 클릭
4. `GoogleService-Info.plist` 다운로드
5. 파일 위치: Flutter 프로젝트 생성 후 아래 경로에 배치
   ```
   [프로젝트]/ios/Runner/GoogleService-Info.plist
   ```
   > Xcode에서 **직접 드래그 앤 드롭**으로 추가해야 합니다 (파일 복사만으로는 안 됨)

- [ ] iOS 앱 등록 완료
- [ ] GoogleService-Info.plist 다운로드 완료
- [ ] Xcode에서 Runner 그룹에 파일 추가 완료

---

## Step 4. APNs 키 등록 (iOS 푸시 알림 필수)

> Apple Developer 계정 필요 (`03_Apple_Developer_설정.md` 먼저 완료)

1. Apple Developer → **Certificates, IDs & Profiles → Keys**
2. **+** 버튼 → Key Name: `MPO FCM Key`
3. **Apple Push Notifications service (APNs)** 체크
4. **Continue → Register → Download** (.p8 파일)
5. Firebase 콘솔 → 프로젝트 설정 → **클라우드 메시징 탭**
6. **APNs 인증 키** 업로드:
   - .p8 파일 업로드
   - Key ID 입력 (Apple Developer에서 확인)
   - Team ID 입력 (Apple Developer 계정 페이지 우측 상단)

- [ ] APNs 키 생성 완료 (.p8 다운로드)
- [ ] Firebase에 APNs 키 등록 완료

---

## Step 5. Windows 앱 등록

1. Firebase 콘솔 홈 → **웹 앱 아이콘** 클릭 (Windows는 웹 앱으로 등록)
2. 앱 닉네임: `Midnight Post Office Windows`
3. Firebase Hosting: 체크 안 해도 됨
4. **앱 등록** 클릭
5. 표시되는 Firebase 설정값 복사해서 메모:
   ```
   apiKey: "..."
   authDomain: "..."
   projectId: "..."
   storageBucket: "..."
   messagingSenderId: "..."
   appId: "..."
   ```

- [ ] Windows(웹) 앱 등록 완료
- [ ] 설정값 메모 완료

---

## Step 6. flutterfire CLI로 firebase_options.dart 생성

> Flutter 프로젝트 생성 후 실행합니다.

```bash
# flutterfire CLI 설치
dart pub global activate flutterfire_cli

# Flutter 프로젝트 루트에서 실행
cd [Flutter 프로젝트 경로]
flutterfire configure
```

실행하면 자동으로 `lib/firebase_options.dart` 파일이 생성됩니다.

- [ ] flutterfire CLI 설치 완료
- [ ] firebase_options.dart 생성 완료

---

## Step 7. FCM 서버 키 확인

Edge Function에서 푸시 알림 발송 시 필요합니다.

1. Firebase 콘솔 → **프로젝트 설정 → 서비스 계정**
2. **새 비공개 키 생성** 클릭 → JSON 파일 다운로드
3. 이 JSON 파일 내용을 Supabase Edge Function 환경변수로 등록:

```bash
supabase secrets set FIREBASE_SERVICE_ACCOUNT='JSON 파일 전체 내용을 한 줄로'
```

> 이 JSON 파일은 절대 GitHub에 올리지 마세요.

- [ ] Firebase 서비스 계정 JSON 다운로드 완료
- [ ] Supabase secrets에 등록 완료

---

## 완료 체크

- [ ] Step 1 — Firebase 프로젝트 생성
- [ ] Step 2 — Android 앱 등록 + google-services.json 배치
- [ ] Step 3 — iOS 앱 등록 + GoogleService-Info.plist 배치
- [ ] Step 4 — APNs 키 등록
- [ ] Step 5 — Windows 앱 등록
- [ ] Step 6 — firebase_options.dart 생성
- [ ] Step 7 — FCM 서버 키 Supabase에 등록
