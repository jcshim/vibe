# 구글 로그인 방명록 만들기 (초보자 가이드)

GitHub + Vercel + Firebase, 세 가지 툴로 구글 로그인이 되는 방명록 웹사이트를 만드는 방법입니다.

---

## 1. GitHub — 코드 저장소 만들기

1. GitHub에서 새 리포지토리(repository)를 하나 만듭니다.
2. 그 안에 `index.html` 파일을 하나 만듭니다. (방명록 웹페이지 코드가 들어갈 파일)

---

## 2. Vercel — 웹사이트로 배포하기

1. Vercel에서 새 프로젝트를 만듭니다.
2. 방금 만든 GitHub 리포지토리 주소를 연결(Import)합니다.
3. 배포가 완료되면 `xxx.vercel.app` 형태의 실제 접속 주소가 생깁니다.

---

## 3. Firebase — 로그인 및 데이터 저장 설정하기

### 3-1. 프로젝트 생성
Firebase 콘솔(console.firebase.google.com)에서 새 프로젝트를 만듭니다.
→ **웹 앱 추가** → `firebaseConfig` 코드가 나오면 복사해서 `index.html`에 붙여넣습니다.

### 3-2. Firestore(DB) 활성화
왼쪽 메뉴 **"데이터베이스 및 스토리지"** → **Firestore Database** → **"데이터베이스 만들기"** 클릭
- 위치는 **서울(asia-northeast3)** 권장
- 처음에는 **테스트 모드**로 시작 (개발 단계에서 편함)

### 3-3. Google 로그인 활성화
왼쪽 메뉴 **"Authentication"** → **로그인 방법** 탭 → **Google** 제공업체를 켭니다.

### 3-4. 승인된 도메인 등록
**Authentication** → **설정** 탭 → **승인된 도메인**에 실제 배포 주소를 추가합니다.
(예: `xxx.vercel.app` — 2단계에서 만든 Vercel 주소)

---

## 확인 순서 요약

| 순서 | 어디서 | 무엇을 |
|---|---|---|
| 1 | GitHub | 리포지토리 + index.html 생성 |
| 2 | Vercel | GitHub 연결 후 배포 |
| 3 | Firebase | 프로젝트 생성 → Firestore 활성화 → Google 로그인 활성화 → 승인된 도메인 등록 |

> 이 4가지(Firebase) 설정이 빠지면 "로그인 중 오류가 발생했습니다" 같은 에러가 뜰 수 있습니다.
