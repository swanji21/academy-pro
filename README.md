# Academy Pro ⚽

**한국어** · [English](#academy-pro-english)

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

<br>

---

# Academy Pro (English)

[한국어](#academy-pro-) · **English**

A mobile-first PWA for running a soccer academy. Manage members, attendance,
lesson schedules, and tuition payment status from a single screen. It runs from
one `index.html` file — no build step required.

## Features

- **Dashboard** — totals for members, attendance, and payments; per-class stats; unpaid members at a glance
- **Members** — add / edit / delete, search & filter (class / level / status), pagination
- **Attendance** — toggle present / late / absent by date and class, mark-all-present
- **Schedule** — add / edit / delete lessons, color-coded
- **Tuition** — per-level fees, paid-status toggle, per-class payment overview
- **Persistence** — all data auto-saved on the device (localStorage)
- **PC ↔ phone realtime sync** — sign in with Firebase to sync across devices on the same account

## Running it

It's a static file — just serve it over HTTP.

```bash
# from the repo folder
python3 -m http.server 8000
# open http://localhost:8000 in a browser
```

It works as-is on any static host (GitHub Pages, Vercel, Netlify, etc.).

> Firebase sign-in can be restricted under `file://` (opening the file directly),
> so serve it via a web server or a host.

## Using it without sync (default)

The app works fully even without Firebase; data is stored **only on that device**.
To share data across multiple devices, set up Firebase as below.

---

## Firebase setup (PC ↔ phone sync)

Sign in with the same account on your PC and phone, and members / attendance /
schedule / tuition data sync in realtime. Each account's data is isolated
(`academies/{uid}`) and only accessible by that account.

### 1. Create a Firebase project

1. Go to the [Firebase console](https://console.firebase.google.com) and **Add project**
2. Enter a project name (e.g. `academy-pro`) → create

### 2. Enable Email/Password sign-in

1. Left menu **Build → Authentication → Get started**
2. **Sign-in method** tab → **Email/Password** → **Enable** and save

### 3. Create a Realtime Database

1. Left menu **Build → Realtime Database → Create Database**
2. Pick a location (e.g. `asia-southeast1`) → choose **Start in locked mode** → create
3. In the **Rules** tab, replace the rules with the following and **Publish**:

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

   > These rules allow read/write only to the signed-in user's own data (`academies/{your uid}`).

### 4. Copy the web app config

1. Project settings (⚙️) → **General** tab → **Your apps** → add a web app (`</>`)
2. Register with a nickname, then copy the `firebaseConfig` values shown

### 5. Paste the config into `index.html`

Fill the `FIREBASE_CONFIG` object in `index.html` with your copied values.
(`databaseURL` must be included; if missing, use the URL at the top of the
Realtime Database page.)

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

### 6. Sign in and use it

1. Open the app, tap the **👤 button** (top right) → **Sign up** to create an account
2. On another device (e.g. your phone), open the app and **sign in with the same email/password**
3. Changes on one device now appear on the other in realtime (the header icon turns ☁️ when syncing)

### Sync status icons

| Icon | Meaning |
| --- | --- |
| 👤 | Signed out (local-only) or Firebase not configured |
| ☁️ | Signed in + syncing in realtime |

---

## iPhone tips

- In Safari, use **Share → Add to Home Screen** to install it as a full-screen app.
- Safe-area insets and the dynamic viewport (`dvh`) keep the bottom tab bar clear of the home indicator / toolbar.

## Tech stack

- Plain HTML/CSS/JavaScript (no framework, no build)
- Data: localStorage + Firebase Authentication + Realtime Database (compat SDK 10.12.2)
- Offline / local fallback supported

## Where data is stored

| Situation | Storage |
| --- | --- |
| Firebase not configured / signed out | Browser localStorage (device-local) |
| Signed in | Firebase Realtime Database `academies/{uid}` + localStorage cache |
