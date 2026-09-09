# Academy Pro ⚽

축구 아카데미 운영을 위한 모바일 우선 PWA. 회원 관리, 출석 체크, 레슨 일정, 수업료 납부 현황을
한 화면에서 관리합니다. 별도 빌드 과정 없이 `index.html` 하나로 동작합니다.

## 주요 기능

- **대시보드** — 전체 회원·출석·납부 현황, 반별 통계, 미납 회원 한눈에 보기
- **회원 관리** — 등록·수정·삭제, 검색·필터(반/레벨/상태), 페이지네이션
- **출석 체크** — 날짜·반별 출석/지각/결석 토글, 전체 출석 처리
- **레슨 일정** — 일정 추가·수정·삭제, 색상 분류
- **수업료** — 레벨별 수업료, 납부 상태 토글, 반별 납부 현황
- **데이터 지속성** — 모든 데이터가 기기에 자동 저장 (localStorage)
- **PC ↔ 폰 실시간 동기화** — Firebase 로그인 시 같은 계정 기기끼리 실시간 동기화

## 실행 방법

정적 파일이라 웹서버로 열기만 하면 됩니다.

```bash
# 저장소 폴더에서
python3 -m http.server 8000
# 브라우저에서 http://localhost:8000 접속
```

GitHub Pages, Vercel, Netlify 등 어떤 정적 호스팅에 올려도 그대로 동작합니다.

> Firebase 로그인 기능은 `file://`(파일 직접 열기)에서는 제한될 수 있으니 웹서버나 호스팅으로 여세요.

## 동기화 없이 쓰기 (기본)

Firebase를 설정하지 않아도 앱은 정상 동작하며, 데이터는 **해당 기기에만** 저장됩니다.
여러 기기에서 데이터를 함께 쓰려면 아래 Firebase 설정이 필요합니다.

---

## Firebase 설정 방법 (PC ↔ 폰 동기화)

같은 계정으로 PC와 휴대폰에 로그인하면 회원·출석·일정·수업료 데이터가 실시간으로 동기화됩니다.
데이터는 계정별로 분리되어(`academies/{uid}`) 본인만 접근할 수 있습니다.

### 1. Firebase 프로젝트 만들기

1. [Firebase 콘솔](https://console.firebase.google.com)에 접속해 **프로젝트 추가**
2. 프로젝트 이름 입력 (예: `academy-pro`) → 생성

### 2. 이메일/비밀번호 로그인 활성화

1. 좌측 메뉴 **빌드 → Authentication → 시작하기**
2. **Sign-in method** 탭 → **이메일/비밀번호** → **사용 설정** 켜고 저장

### 3. Realtime Database 만들기

1. 좌측 메뉴 **빌드 → Realtime Database → 데이터베이스 만들기**
2. 위치 선택 (예: `asia-southeast1`) → **잠금 모드로 시작** 선택 후 생성
3. **규칙(Rules)** 탭에서 아래 내용으로 교체하고 **게시**:

   ```json
   {
     "rules": {
       "academies": {
         "$uid": {
           ".read": "auth != null && auth.uid === $uid",
           ".write": "auth != null && auth.uid === $uid"
         }
       }
     }
   }
   ```

   > 이 규칙은 로그인한 본인의 데이터(`academies/{내 uid}`)에만 읽기/쓰기를 허용합니다.

### 4. 웹 앱 설정값(config) 복사

1. 프로젝트 설정(⚙️) → **일반** 탭 → **내 앱** → 웹 앱 추가(`</>`)
2. 앱 닉네임 입력 후 등록하면 나오는 `firebaseConfig` 값을 복사

### 5. `index.html`에 설정값 붙여넣기

`index.html` 안의 `FIREBASE_CONFIG` 객체를 복사한 값으로 채웁니다.
(`databaseURL`이 포함되어야 하며, 없으면 Realtime Database 페이지 상단 URL을 넣으세요.)

```js
const FIREBASE_CONFIG = {
  apiKey:            "AIza...",
  authDomain:        "academy-pro.firebaseapp.com",
  databaseURL:       "https://academy-pro-default-rtdb.asia-southeast1.firebasedatabase.app",
  projectId:         "academy-pro",
  storageBucket:     "academy-pro.appspot.com",
  messagingSenderId: "1234567890",
  appId:             "1:1234567890:web:abcdef123456"
};
```

### 6. 로그인 후 사용

1. 앱을 열고 우측 상단 **👤 버튼** → **회원가입**으로 계정 생성
2. 휴대폰 등 다른 기기에서 앱을 열고 **같은 이메일/비밀번호로 로그인**
3. 이제 한쪽에서 바꾸면 다른 기기에 실시간으로 반영됩니다 (헤더 아이콘이 ☁️로 바뀌면 동기화 중)

### 동기화 상태 아이콘

| 아이콘 | 의미 |
| --- | --- |
| 👤 | 로그아웃 상태 (로컬 저장만) 또는 Firebase 미설정 |
| ☁️ | 로그인 + 실시간 동기화 중 |

---

## 아이폰 사용 팁

- Safari에서 **공유 → 홈 화면에 추가**로 설치하면 전체 화면 앱처럼 실행됩니다.
- 하단 탭바가 홈 인디케이터/툴바에 가리지 않도록 safe-area와 동적 뷰포트(`dvh`)를 적용했습니다.

## 기술 스택

- 순수 HTML/CSS/JavaScript (프레임워크·빌드 없음)
- 데이터: localStorage + Firebase Authentication + Realtime Database (compat SDK 10.12.2)
- 오프라인/로컬 폴백 지원

## 데이터 저장 위치

| 상황 | 저장 위치 |
| --- | --- |
| Firebase 미설정 / 로그아웃 | 브라우저 localStorage (기기 로컬) |
| 로그인 | Firebase Realtime Database `academies/{uid}` + localStorage 캐시 |
