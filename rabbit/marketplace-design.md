# 🛒 중고 거래 플랫폼 완전 설계도

## 📦 기술 스택
- **Frontend**: React → Netlify
- **Backend**: Netlify Functions (서버리스)
- **Realtime Chat**: Firestore 실시간 리스너
- **Database**: Firebase Firestore
- **Storage**: Firebase Storage
- **Auth**: Firebase Authentication
- **배포**: Netlify (프론트엔드 + 백엔드 통합)

---

## 🤖 Sub Agent 역할 분담 및 작업 순서

### Phase 1: 기초 설정 (병렬 작업 가능)
- **Agent A - Firebase Setup**: Firebase 프로젝트 생성 및 설정
  - Firestore 데이터베이스 생성
  - Storage 버킷 생성
  - Authentication 활성화
  - Firebase SDK 설정 파일 생성

- **Agent B - Project Init**: React + Netlify Functions 프로젝트 초기화
  - Create React App 설치
  - 폴더 구조 생성
  - 필요한 패키지 설치
  - Git 저장소 초기화

### Phase 2: 핵심 기능 (의존성 기반 순차/병렬)
- **Agent A - Auth System**: Firebase Authentication 통합 (우선순위 높음)
  - AuthContext 구현
  - Login/Register 컴포넌트
  - PrivateRoute 구현
  - Firebase Auth 연동

- **Agent B - Product CRUD**: 상품 기능 (Agent A 완료 후 시작)
  - ProductList, ProductCard, ProductDetail 컴포넌트
  - ProductForm (상품 등록/수정)
  - Netlify Functions (products-get, products-create, products-update, products-delete)
  - Firestore CRUD 로직

- **Agent C - Image Upload**: Firebase Storage 통합 (Agent A와 병렬 가능)
  - ImageUpload 컴포넌트
  - Firebase Storage 업로드 로직
  - 이미지 미리보기 기능
  - 다중 이미지 처리 (최대 5개)

- **Agent D - Chat System**: Firestore 실시간 채팅 (Agent A 완료 후)
  - ChatList, ChatRoom, MessageBubble 컴포넌트
  - useChat 커스텀 훅 (Firestore 실시간 리스너)
  - chats-get, messages-create Functions
  - 읽음/안읽음 상태 관리

### Phase 3: 고급 기능 (병렬 작업 가능)
- **Agent A - Points System**: 포인트 거래 로직
  - PointsBalance 컴포넌트
  - transactions-create Function
  - 포인트 이동 로직
  - 거래 내역 조회

- **Agent B - UI/UX Polish**: 스타일링 및 사용성 개선
  - 반응형 디자인
  - 로딩 상태 표시
  - 에러 메시지 개선
  - 전체적인 스타일링

### Phase 4: 배포
- **Agent A - Netlify Deploy**: 통합 배포
  - netlify.toml 설정
  - 환경변수 설정
  - GitHub 연동
  - 자동 배포 확인

---

## 📐 일관성 유지 규칙

### 1. 네이밍 컨벤션
```javascript
// 파일명
// - React 컴포넌트: PascalCase.jsx (예: ProductCard.jsx)
// - 유틸리티/서비스: camelCase.js (예: firebase.js)
// - Netlify Functions: kebab-case.js (예: products-get.js)

// 컴포넌트명: PascalCase
export default function ProductCard() { }

// 함수/변수: camelCase
const fetchProducts = () => { }
const userId = "123"

// 상수: UPPER_SNAKE_CASE
const MAX_IMAGE_COUNT = 5
const API_BASE_URL = "/.netlify/functions"

// Firestore 컬렉션: 소문자 복수형
// users, products, chats, messages, transactions

// Netlify Functions 엔드포인트
// /.netlify/functions/products-get
// /.netlify/functions/products-create
```

### 2. 폴더 구조 규칙
```
- 기능별로 폴더 분리 (auth, products, chat, user, common)
- 각 폴더는 독립적으로 동작 가능하게 구성
- 관련된 컴포넌트는 같은 폴더에 배치
- 공통 컴포넌트는 common 폴더
- 커스텀 훅은 hooks 폴더
- 서비스 로직은 services 폴더
```

### 3. API 응답 형식 (모든 Agent가 준수해야 함)
```javascript
// 성공 응답
{
  "success": true,
  "data": {
    // 실제 데이터
  },
  "message": "Success message"
}

// 에러 응답
{
  "success": false,
  "error": {
    "code": "ERROR_CODE",
    "message": "User-friendly error message"
  }
}

// HTTP 상태 코드
// 200: 성공
// 201: 생성 성공
// 400: 잘못된 요청
// 401: 인증 실패
// 403: 권한 없음
// 404: 찾을 수 없음
// 500: 서버 에러
```

### 4. 에러 핸들링 표준
```javascript
// Frontend: try-catch + 사용자 친화적 메시지
try {
  const response = await api.getProducts()
  setProducts(response.data)
} catch (error) {
  console.error('Failed to fetch products:', error)
  alert('상품을 불러오는데 실패했습니다.')
}

// Netlify Functions: 일관된 에러 응답
exports.handler = async (event) => {
  try {
    // 로직
  } catch (error) {
    return {
      statusCode: 500,
      body: JSON.stringify({
        success: false,
        error: {
          code: 'INTERNAL_ERROR',
          message: error.message
        }
      })
    }
  }
}
```

### 5. 환경변수 네이밍
```bash
# Frontend (.env)
REACT_APP_FIREBASE_API_KEY=xxx
REACT_APP_FIREBASE_AUTH_DOMAIN=xxx
REACT_APP_FIREBASE_PROJECT_ID=xxx
REACT_APP_FIREBASE_STORAGE_BUCKET=xxx

# Netlify Functions (Netlify 대시보드에서 설정)
FIREBASE_PROJECT_ID=xxx
FIREBASE_CLIENT_EMAIL=xxx
FIREBASE_PRIVATE_KEY=xxx
```

### 6. Git 커밋 메시지 규칙
```
feat: 새로운 기능 추가
fix: 버그 수정
style: UI/스타일 변경
refactor: 코드 리팩토링
docs: 문서 수정
deploy: 배포 관련 설정
chore: 기타 작업

예시:
feat: 상품 목록 조회 기능 추가
fix: 이미지 업로드 오류 수정
style: 상품 카드 디자인 개선
```

---

## 📁 프로젝트 구조

```
marketplace-app/
├── public/
│   ├── index.html
│   └── _redirects                   # Netlify SPA 라우팅 설정
│
├── src/
│   ├── components/
│   │   ├── auth/
│   │   │   ├── Login.jsx           # 로그인 폼
│   │   │   ├── Register.jsx        # 회원가입 폼
│   │   │   └── PrivateRoute.jsx    # 인증된 사용자만 접근 가능한 라우트
│   │   │
│   │   ├── products/
│   │   │   ├── ProductList.jsx     # 상품 목록
│   │   │   ├── ProductCard.jsx     # 상품 카드
│   │   │   ├── ProductDetail.jsx   # 상품 상세
│   │   │   └── ProductForm.jsx     # 상품 등록/수정 폼
│   │   │
│   │   ├── chat/
│   │   │   ├── ChatList.jsx        # 채팅 목록
│   │   │   ├── ChatRoom.jsx        # 채팅방
│   │   │   └── MessageBubble.jsx   # 메시지 말풍선
│   │   │
│   │   ├── user/
│   │   │   ├── Profile.jsx         # 사용자 프로필
│   │   │   └── PointsBalance.jsx   # 포인트 잔액 표시
│   │   │
│   │   └── common/
│   │       ├── Header.jsx          # 헤더/네비게이션
│   │       ├── ImageUpload.jsx     # 이미지 업로드 컴포넌트
│   │       └── Loading.jsx         # 로딩 스피너
│   │
│   ├── contexts/
│   │   └── AuthContext.jsx         # 인증 상태 관리
│   │
│   ├── services/
│   │   ├── firebase.js             # Firebase SDK 초기화
│   │   └── api.js                  # Netlify Functions 호출
│   │
│   ├── hooks/
│   │   ├── useAuth.js              # 인증 관련 훅
│   │   ├── useChat.js              # 채팅 (Firestore 실시간 리스너)
│   │   └── useProducts.js          # 상품 관련 훅
│   │
│   ├── pages/
│   │   ├── Home.jsx                # 홈 페이지 (상품 목록)
│   │   ├── ProductPage.jsx         # 상품 상세 페이지
│   │   ├── ChatPage.jsx            # 채팅 페이지
│   │   └── ProfilePage.jsx         # 프로필 페이지
│   │
│   ├── App.jsx                     # 메인 앱 컴포넌트
│   └── index.js                    # 엔트리 포인트
│
├── netlify/
│   └── functions/                  # Netlify Functions (백엔드)
│       ├── products-get.js         # 상품 목록/상세 조회
│       ├── products-create.js      # 상품 생성
│       ├── products-update.js      # 상품 수정
│       ├── products-delete.js      # 상품 삭제
│       ├── users-get.js            # 사용자 정보 조회
│       ├── users-update.js         # 사용자 정보 수정
│       ├── chats-get.js            # 채팅 목록 조회
│       ├── messages-create.js      # 메시지 생성
│       ├── transactions-create.js  # 거래 생성
│       └── utils/
│           ├── firebaseAdmin.js    # Firebase Admin SDK 초기화
│           ├── auth.js             # 인증 미들웨어
│           └── response.js         # 응답 헬퍼 함수
│
├── netlify.toml                    # Netlify 배포 설정
├── .env                            # 로컬 환경변수
├── .env.production                 # 프로덕션 환경변수
├── .gitignore
├── package.json
└── README.md
```

---

## 🗄️ Firestore 데이터 구조

### users 컬렉션
```javascript
users/{userId}/
{
  email: "user@example.com",          // 이메일
  displayName: "홍길동",               // 표시 이름
  photoURL: "https://...",            // 프로필 사진 URL
  points: 10000,                      // 포인트 (초기값 10000)
  createdAt: Timestamp                // 생성일시
}
```

### products 컬렉션
```javascript
products/{productId}/
{
  title: "아이폰 15 Pro",              // 상품 제목
  description: "깨끗한 상태입니다.",   // 상품 설명
  price: 1000000,                     // 가격
  images: [                           // 이미지 URL 배열 (최대 5개)
    "https://storage.../image1.jpg",
    "https://storage.../image2.jpg"
  ],
  sellerId: "user123",                // 판매자 ID
  status: "available",                // 상태: available, sold, reserved
  category: "전자제품",                // 카테고리
  location: "서울 강남구",             // 거래 희망 지역
  createdAt: Timestamp,               // 생성일시
  updatedAt: Timestamp                // 수정일시
}
```

### chats 컬렉션
```javascript
chats/{chatId}/
{
  participants: ["user123", "user456"], // 참여자 ID 배열 (2명)
  productId: "product789",              // 관련 상품 ID
  lastMessage: "네 좋습니다",            // 마지막 메시지
  lastMessageAt: Timestamp,             // 마지막 메시지 시간
  createdAt: Timestamp                  // 채팅방 생성일시
}
```

### messages 서브컬렉션
```javascript
messages/{chatId}/messages/{messageId}/
{
  senderId: "user123",                 // 발신자 ID
  text: "안녕하세요",                   // 메시지 내용
  timestamp: Timestamp,                // 메시지 전송 시간
  read: false                          // 읽음 여부
}
```

### transactions 컬렉션
```javascript
transactions/{transactionId}/
{
  fromUserId: "user123",               // 포인트 보내는 사람
  toUserId: "user456",                 // 포인트 받는 사람
  amount: 1000000,                     // 거래 금액
  productId: "product789",             // 관련 상품 ID
  status: "completed",                 // 상태: pending, completed, failed
  createdAt: Timestamp                 // 거래 생성일시
}
```

---

## 🔧 Netlify Functions API 엔드포인트

### 상품 관련
```
GET    /.netlify/functions/products-get
       - Query: ?productId={id} (특정 상품 조회)
       - Query: 없음 (전체 상품 목록 조회)

POST   /.netlify/functions/products-create
       - Body: { title, description, price, images, category, location }

PUT    /.netlify/functions/products-update
       - Body: { productId, title, description, price, images, status, ... }

DELETE /.netlify/functions/products-delete
       - Body: { productId }
```

### 사용자 관련
```
GET    /.netlify/functions/users-get
       - Query: ?userId={id}

PUT    /.netlify/functions/users-update
       - Body: { userId, displayName, photoURL, ... }
```

### 채팅 관련
```
GET    /.netlify/functions/chats-get
       - Query: ?userId={id} (사용자의 채팅 목록)

POST   /.netlify/functions/messages-create
       - Body: { chatId, senderId, text }
```

### 거래 관련
```
POST   /.netlify/functions/transactions-create
       - Body: { fromUserId, toUserId, amount, productId }
```

---

## 📦 필요한 패키지

### package.json
```json
{
  "name": "marketplace-app",
  "version": "1.0.0",
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.20.0",
    "firebase": "^10.7.0",
    "axios": "^1.6.0"
  },
  "devDependencies": {
    "netlify-cli": "^17.0.0",
    "firebase-admin": "^12.0.0"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "eject": "react-scripts eject",
    "dev": "netlify dev"
  }
}
```

---

## 🚀 Netlify 배포 설정

### netlify.toml
```toml
[build]
  command = "npm run build"
  functions = "netlify/functions"
  publish = "build"

# API 라우팅
[[redirects]]
  from = "/api/*"
  to = "/.netlify/functions/:splat"
  status = 200

# SPA 라우팅 (React Router)
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

# 환경변수 (Netlify 대시보드에서 설정)
[build.environment]
  NODE_VERSION = "18"
```

### public/_redirects
```
/api/*  /.netlify/functions/:splat  200
/*      /index.html                  200
```

---

## 🔥 실시간 채팅 구현 방식

Socket.io 대신 **Firestore 실시간 리스너**를 사용합니다.

### useChat.js 커스텀 훅 예시
```javascript
import { useEffect, useState } from 'react'
import { collection, query, orderBy, onSnapshot } from 'firebase/firestore'
import { db } from '../services/firebase'

export function useChat(chatId) {
  const [messages, setMessages] = useState([])
  const [loading, setLoading] = useState(true)

  useEffect(() => {
    if (!chatId) return

    const messagesRef = collection(db, 'messages', chatId, 'messages')
    const q = query(messagesRef, orderBy('timestamp', 'asc'))

    // 실시간 리스너 등록
    const unsubscribe = onSnapshot(q, (snapshot) => {
      const newMessages = snapshot.docs.map(doc => ({
        id: doc.id,
        ...doc.data()
      }))
      setMessages(newMessages)
      setLoading(false)
    })

    // 컴포넌트 언마운트 시 리스너 해제
    return () => unsubscribe()
  }, [chatId])

  return { messages, loading }
}
```

### 장점
- 서버리스 환경에서 완벽하게 작동
- WebSocket 연결 관리 불필요
- Firebase에서 자동으로 동기화
- Netlify Functions와 호환성 100%

---

## 📝 단계별 구현 계획

### Step 1: 프로젝트 초기화
```bash
# React 앱 생성
npx create-react-app marketplace-app
cd marketplace-app

# 필요한 패키지 설치
npm install react-router-dom firebase axios

# 개발 도구 설치
npm install --save-dev netlify-cli firebase-admin

# 폴더 구조 생성
mkdir -p src/components/{auth,products,chat,user,common}
mkdir -p src/{contexts,services,hooks,pages}
mkdir -p netlify/functions/utils
```

### Step 2: Firebase 설정
1. Firebase Console에서 프로젝트 생성
2. Firestore 데이터베이스 활성화
3. Storage 활성화
4. Authentication 활성화 (이메일/비밀번호, Google 등)
5. Firebase 설정 정보 복사
6. `src/services/firebase.js` 파일에 설정

### Step 3: 인증 시스템 (Agent A)
- AuthContext 구현
- Login/Register 컴포넌트
- PrivateRoute 구현
- Firebase Auth 연동

### Step 4: 상품 CRUD (Agent B)
- ProductList, ProductCard, ProductDetail 컴포넌트
- ProductForm (등록/수정)
- Netlify Functions 구현
- Firestore CRUD 연동

### Step 5: 이미지 업로드 (Agent C)
- ImageUpload 컴포넌트
- Firebase Storage 업로드
- 이미지 미리보기
- 다중 이미지 처리

### Step 6: 실시간 채팅 (Agent D)
- ChatList, ChatRoom 컴포넌트
- useChat 훅 (Firestore 리스너)
- 메시지 전송 기능
- 읽음/안읽음 표시

### Step 7: 포인트 시스템 (Agent A)
- PointsBalance 컴포넌트
- 거래 생성 Function
- 포인트 이동 로직
- 거래 내역 조회

### Step 8: UI/UX 개선 (Agent B)
- 반응형 디자인
- 로딩 상태
- 에러 메시지
- 전체 스타일링

### Step 9: Netlify 배포 (Agent A)
1. GitHub 저장소 생성 및 푸시
2. Netlify 대시보드에서 사이트 생성
3. GitHub 저장소 연동
4. 환경변수 설정
5. 자동 배포 확인

---

## ✅ Netlify 배포 장점

1. **통합 관리**: 프론트엔드 + 백엔드를 하나의 플랫폼에서 관리
2. **자동 HTTPS**: SSL 인증서 자동 발급 및 갱신
3. **CI/CD**: Git push만으로 자동 빌드 및 배포
4. **무료 티어**: 개인 프로젝트에 충분한 무료 할당량
5. **서버리스**: 트래픽에 따라 자동 스케일링
6. **간편한 설정**: netlify.toml 하나로 모든 설정 관리
7. **환경변수 관리**: 대시보드에서 쉽게 관리

---

## 🎯 핵심 기능 요약

1. **사용자 인증**: Firebase Authentication (이메일/소셜 로그인)
2. **상품 게시판**: CRUD + 이미지 업로드 + 검색/필터
3. **실시간 채팅**: Firestore 실시간 리스너 (1:1 채팅)
4. **포인트 시스템**: 거래 시 자동 포인트 이동
5. **이미지 관리**: Firebase Storage (최대 5장)
6. **반응형 디자인**: 모바일/태블릿/데스크톱 지원

---

## 📌 주의사항

### Agent 작업 시 확인사항
1. **의존성 확인**: 다른 Agent의 작업이 완료되어야 하는지 확인
2. **네이밍 규칙**: 파일명, 변수명 등 규칙 준수
3. **API 응답 형식**: 일관된 형식 사용
4. **에러 처리**: 모든 비동기 작업에 try-catch 적용
5. **환경변수**: 민감한 정보는 환경변수로 관리
6. **커밋 메시지**: 규칙에 맞는 커밋 메시지 작성

### 보안 고려사항
1. Firebase Admin SDK private key는 절대 클라이언트에 노출 금지
2. Firestore Security Rules 설정 필수
3. 인증된 사용자만 특정 작업 수행 가능하도록 제한
4. 이미지 업로드 시 파일 크기 및 형식 검증

### 성능 최적화
1. Firestore 쿼리 최적화 (인덱스 생성)
2. 이미지 압축 및 리사이징
3. React 컴포넌트 메모이제이션
4. 불필요한 리렌더링 방지

---

## 🚀 시작하기

```bash
# 1. 프로젝트 클론
git clone <repository-url>
cd marketplace-app

# 2. 패키지 설치
npm install

# 3. 환경변수 설정
# .env 파일 생성 후 Firebase 설정 입력

# 4. 로컬 개발 서버 실행
npm run dev
# 또는
netlify dev

# 5. 빌드
npm run build

# 6. 배포
netlify deploy --prod
```

---

## 📞 문의 및 지원

프로젝트 진행 중 문제가 발생하면:
1. 이 문서의 규칙을 먼저 확인
2. Firebase 콘솔에서 에러 로그 확인
3. Netlify 대시보드에서 배포 로그 확인
4. Agent 간 작업 충돌이 없는지 확인
