# 배포 전 체크리스트 (Deployment Checklist)

## 🔴 긴급 (Critical) - 반드시 수정 필요

### 1. API 엔드포인트 설정
**파일**: `Record-FE/src/services/api/client.ts`
- **현재**: 프로덕션 URL이 `https://api.ticketbook.app`로 하드코딩됨
- **문제**: 실제 배포 서버 URL과 다를 수 있음
- **해결책**: 
  - 환경 변수 사용 (`react-native-dotenv` 이미 설치됨)
  - 또는 빌드 시점에 주입

```typescript
// .env.production 파일 생성 필요
API_BASE_URL=https://your-actual-api-domain.com
```

### 2. Android Release 빌드 서명 설정
**파일**: `Record-FE/android/app/build.gradle`
- **현재**: Release 빌드도 debug keystore 사용 (103번째 줄)
- **문제**: 프로덕션 앱은 반드시 별도의 release keystore 필요
- **해결책**:
  1. Release keystore 생성
  2. `android/app/release.keystore` 파일 생성
  3. `build.gradle`에 release signingConfig 추가
  4. keystore 정보는 환경 변수로 관리 (절대 커밋 금지)

```gradle
signingConfigs {
    release {
        storeFile file('release.keystore')
        storePassword System.getenv("KEYSTORE_PASSWORD")
        keyAlias System.getenv("KEY_ALIAS")
        keyPassword System.getenv("KEY_PASSWORD")
    }
}
```

### 3. Android 보안 설정
**파일**: `Record-FE/android/app/src/main/AndroidManifest.xml`
- **문제**: `android:usesCleartextTraffic="true"` (17번째 줄)
- **해결책**: 프로덕션에서는 HTTPS만 사용하도록 변경
  - 개발 환경에서만 cleartext 허용
  - Network Security Config 파일 생성

### 4. 백엔드 설정 확인
**파일**: `Record-BE/Record-BE/src/main/resources/application.yml`
- **확인 필요**:
  - `app.base-url`: 실제 도메인으로 변경 (현재: `https://record.example.com`)
  - `app.mail.from`: 실제 이메일 주소로 변경
  - 환경 변수 설정 확인 (DB, OpenAI API Key 등)

### 5. 이미지 URL 해결
**파일**: `Record-FE/src/utils/resolveImageUrl.ts`
- **현재**: API_BASE_URL 사용 중 (적절함)
- **확인**: S3 또는 CDN URL이 올바르게 설정되어 있는지 확인

---

## 🟡 중요 (Important) - 배포 전 권장

### 6. 프로덕션 로깅 제거/최소화
**파일**: 여러 파일
- **문제**: `console.log`, `console.error`가 프로덕션에도 출력됨
- **해결책**: 
  - 로깅 라이브러리 도입 (예: `react-native-logs`)
  - 또는 `__DEV__` 체크로 프로덕션에서 제거
  - 에러 리포팅 서비스 연동 (Sentry, Crashlytics)

**영향받는 파일들**:
- `Record-FE/src/utils/resolveImageUrl.ts` (23번째 줄)
- `Record-FE/src/utils/ticketHelpers.ts` (134, 137번째 줄)
- `Record-FE/src/services/auth/authService.ts` (84, 85번째 줄)
- `Record-FE/src/contexts/ThemeContext.tsx` (43, 64번째 줄)

### 7. 에러 리포팅 서비스 연동
**파일**: `Record-FE/src/components/ErrorBoundary.tsx`
- **현재**: 주석으로만 언급됨 (60-61번째 줄)
- **해결책**: 
  - Sentry 또는 Firebase Crashlytics 연동
  - 프로덕션 에러 자동 수집

### 8. 앱 버전 관리
**파일**: 
- `Record-FE/package.json` (version: 0.0.1)
- `Record-FE/android/app/build.gradle` (versionCode: 1, versionName: "1.0")
- `Record-FE/src/pages/my-page/SettingsPage.tsx` (252번째 줄: "버전 1.0.0")

- **문제**: 버전이 일치하지 않음
- **해결책**: 
  - 단일 소스에서 버전 관리
  - 빌드 시 자동 동기화

### 9. ProGuard 설정 (Android)
**파일**: `Record-FE/android/app/build.gradle`
- **현재**: `enableProguardInReleaseBuilds = false` (60번째 줄)
- **권장**: 프로덕션 빌드에서 활성화하여 앱 크기 감소 및 난독화
- **주의**: ProGuard 규칙 추가 필요 (`proguard-rules.pro`)

### 10. iOS 설정 확인
**확인 필요**:
- `Record-FE/ios/TicketBookApp/Info.plist` - 권한 설정
- Bundle Identifier 확인
- Code Signing 설정
- App Store Connect 설정

---

## 🟢 개선 권장 (Recommended)

### 11. 환경 변수 관리
- `.env.development`, `.env.production` 파일 생성
- `.gitignore`에 `.env*` 추가 확인
- 빌드 스크립트에 환경별 설정 적용

### 12. 성능 최적화
- 이미지 최적화 (압축, WebP 사용)
- 번들 크기 분석
- 메모리 누수 확인
- 리렌더링 최적화

### 13. 보안 강화
- API 키 하드코딩 확인 (없어야 함)
- 토큰 저장 방식 검토 (현재 AsyncStorage 사용 중)
- HTTPS 강제
- Certificate Pinning 고려

### 14. 테스트
- [ ] 단위 테스트 실행
- [ ] 통합 테스트 실행
- [ ] E2E 테스트 (선택)
- [ ] 실제 기기에서 테스트
- [ ] 다양한 OS 버전 테스트
- [ ] 네트워크 오류 시나리오 테스트

### 15. 문서화
- [ ] API 문서 확인
- [ ] 배포 가이드 작성
- [ ] 롤백 계획 수립
- [ ] 모니터링 설정

### 16. 앱 스토어 준비
**Android (Google Play)**:
- [ ] 앱 아이콘 (모든 해상도)
- [ ] 스플래시 스크린
- [ ] 스크린샷
- [ ] 앱 설명
- [ ] 개인정보 처리방침 URL
- [ ] Privacy Policy

**iOS (App Store)**:
- [ ] 앱 아이콘
- [ ] Launch Screen
- [ ] 스크린샷
- [ ] 앱 설명
- [ ] 개인정보 처리방침
- [ ] App Store Connect 설정

### 17. 모니터링 및 분석
- [ ] Crashlytics 설정
- [ ] Analytics 설정 (선택)
- [ ] 성능 모니터링
- [ ] 사용자 피드백 수집 방법

### 18. 백엔드 배포 확인
- [ ] 프로덕션 데이터베이스 설정
- [ ] 환경 변수 설정
- [ ] CORS 설정 확인
- [ ] Rate Limiting 설정
- [ ] 백업 설정
- [ ] 로그 수집 설정

---

## 📋 배포 전 최종 체크리스트

### 필수 항목
- [ ] API 엔드포인트가 실제 프로덕션 URL로 설정됨
- [ ] Android Release keystore 생성 및 설정
- [ ] iOS Code Signing 설정 완료
- [ ] 백엔드 프로덕션 환경 설정 완료
- [ ] 데이터베이스 마이그레이션 완료
- [ ] 환경 변수 모두 설정됨
- [ ] 보안 설정 확인 (HTTPS, cleartext 제거)
- [ ] 앱 버전 일치 확인

### 권장 항목
- [ ] 에러 리포팅 서비스 연동
- [ ] 프로덕션 로깅 최소화
- [ ] ProGuard 활성화 (Android)
- [ ] 성능 테스트 완료
- [ ] 보안 검토 완료
- [ ] 앱 스토어 자료 준비

### 배포 후
- [ ] 모니터링 대시보드 확인
- [ ] 에러 로그 확인
- [ ] 사용자 피드백 수집
- [ ] 성능 지표 확인

---

## 🚨 즉시 수정 필요한 항목 우선순위

1. **Android Release Keystore 설정** (보안)
2. **API 엔드포인트 환경 변수화** (기능)
3. **Android Cleartext Traffic 제거** (보안)
4. **백엔드 base-url 설정** (기능)
5. **에러 리포팅 서비스 연동** (운영)

---

## 📝 참고 자료

- [React Native 배포 가이드](https://reactnative.dev/docs/signed-apk-android)
- [Android 보안 설정](https://developer.android.com/training/articles/security-config)
- [iOS 배포 가이드](https://reactnative.dev/docs/publishing-to-app-store)

