---
name: ai-dlc-fe-node-setup
description: AI-DLC 개발단계(프론트엔드-React) 스킬. Node.js/Express BFF 또는 목업 API 서버 초기 설정을 생성한다. "Node.js 서버 만들어줘", "Express 서버 초기화", "BFF 서버 만들어줘", "Mock API 서버 생성", "Node.js/Express 프로젝트 생성", "Express 백엔드 만들어줘", "노드 서버 셋업", "프록시 서버 만들어줘" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Node.js/Express BFF·Mock 서버 초기 설정

React 앱과 연동하는 **BFF(Backend For Frontend)** 또는 **Mock API 서버**를 Node.js/Express + TypeScript로 초기화한다. 백엔드 Spring Boot가 준비되지 않은 개발 초기 단계, 또는 API 집계·프록시 레이어가 필요한 경우 사용한다.
공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "Node.js 서버 만들어줘", "Express 서버 초기화", "BFF 서버 만들어줘"
- "Mock API 서버 생성", "Node.js/Express 프로젝트 생성", "Express 백엔드 만들어줘"
- "노드 서버 셋업", "프록시 서버 만들어줘", "목업 서버 만들어줘"

---

## 입력

### 필수
- 서버 목적: **BFF**(실제 백엔드 프록시·집계) 또는 **Mock**(더미 데이터 응답)
- 포트 번호 (기본값: 4000)

### 선택
- 대상 백엔드 URL (BFF 용도일 때 프록시 타겟, 예: `http://localhost:8080`)
- API 설계서(operationId) — Mock 라우트 자동 생성 시

---

## 분석 절차

### 1단계: 서버 목적 파악
- **BFF**: 클라이언트-백엔드 중간 레이어. 인증 토큰 교환, API 응답 변환, 여러 백엔드 API 집계.
- **Mock**: 백엔드 없이 프론트엔드 개발용 더미 응답 서버. 실제 API 스펙과 동일한 응답 형식 사용.

### 2단계: 의존성 구성

**dependencies (공통)**: `express`, `cors`, `morgan`, `dotenv`, `helmet`
**BFF 추가**: `http-proxy-middleware`
**devDependencies**: `typescript`, `ts-node-dev`, `@types/express`, `@types/cors`, `@types/morgan`, `@types/node`

### 3단계: 디렉터리 구조 생성
```
src/
├── app.ts                    # Express 앱 인스턴스 + 미들웨어
├── server.ts                 # HTTP 서버 시작 (listen)
├── routes/
│   └── index.ts              # 라우터 집합
├── middleware/
│   └── errorHandler.ts       # 공통 에러 핸들러
└── types/
    └── api-response.ts       # ApiResponse<T> 타입
```

### 4단계: 핵심 파일 생성
- `src/app.ts`: Express 앱, cors + morgan + helmet + express.json 미들웨어
- `src/server.ts`: `app.listen(PORT)` 실행
- `src/routes/index.ts`: 도메인별 라우터 등록
- `src/middleware/errorHandler.ts`: `next(err)` 패턴 전역 에러 핸들러
- `.env.example`: `PORT`, `TARGET_API_URL` (BFF 용도)
- `tsconfig.json`: Node.js용 (`module: commonjs`, `outDir: dist`)

### 5단계: Mock 라우트 생성 (Mock 목적인 경우)
API 설계서의 operationId를 기반으로 더미 라우트 자동 생성.
```
GET  /api/users        → [{ userId: 1, userNm: "홍길동" }]
POST /api/users        → { userId: 2, message: "등록 완료" }
GET  /api/users/:id    → { userId: 1, userNm: "홍길동" }
```

### 6단계: BFF 프록시 설정 (BFF 목적인 경우)
- `http-proxy-middleware`로 `/api/*` → 백엔드 서버 프록시
- 토큰 검증·교환 로직 추가

---

## 생성 원칙

- **응답 형식**: 백엔드 Spring Boot와 동일한 `{ code, message, data }` ApiResponse 래퍼
- **CORS**: React dev server 포트(`http://localhost:3000`) 허용
- **에러 처리**: 중앙 에러 핸들러 미들웨어, HTTP 상태코드 매핑
- **보안**: `helmet()` 기본 적용

---

## 산출물

| 파일 | 설명 |
|:---|:---|
| `package.json` | Node.js 의존성·스크립트 (dev, build, start) |
| `tsconfig.json` | TypeScript(Node용) 설정 |
| `src/app.ts` | Express 앱 + 미들웨어 |
| `src/server.ts` | HTTP 서버 진입점 |
| `src/routes/index.ts` | 라우터 집합 |
| `src/middleware/errorHandler.ts` | 전역 에러 핸들러 |
| `src/types/api-response.ts` | ApiResponse<T> 타입 |
| `.env.example` | 환경 변수 예시 |

template.md에서 각 파일의 기본 코드 골격을 참조한다.
