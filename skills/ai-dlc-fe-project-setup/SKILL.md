---
name: ai-dlc-fe-project-setup
description: AI-DLC 개발단계(프론트엔드-React) 스킬. React/Vite 프로젝트 초기 설정 파일 일체를 생성한다. "React 프로젝트 만들어줘", "Vite 프로젝트 초기화", "React/Vite 설정해줘", "프론트엔드 프로젝트 생성", "React 앱 시작해줘", "Vite 셋업", "리액트 프로젝트 초기화", "프론트엔드 초기 설정" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC React/Vite 프로젝트 초기 설정

설계단계 산출물(화면설계서·API설계서)을 기반으로 React/Vite + TypeScript 프로젝트 초기 설정 파일 일체를 생성한다.
공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "React 프로젝트 만들어줘", "Vite 프로젝트 초기화", "React/Vite 설정해줘"
- "프론트엔드 프로젝트 생성", "React 앱 시작해줘", "리액트 프로젝트 셋업"
- "Vite 설정 만들어줘", "프론트엔드 초기 설정"

---

## 입력

### 필수
- 프로젝트명 (디렉터리명·package.json name)
- 기술 스택 선택:
  - TypeScript 5.x strict (기본값, 고정)
  - React 18.x (기본값)
  - 상태 관리: Zustand (기본값) / Redux Toolkit (선택)
  - API 서버 URL (vite.config.ts proxy 설정)

### 선택
- 화면설계서(SCR-NNN) — 라우팅 경로 추출 시 활용
- 팀 ESLint 규칙 파일

---

## 분석 절차

### 1단계: 기술 스택 파악
사용자가 언급한 스택을 정리하고, 미언급 항목은 기본값 적용.

### 2단계: 의존성 구성 (package.json)

**dependencies**:
- 코어: `react`, `react-dom`, `react-router-dom`
- 상태 관리: `zustand` (기본) 또는 `@reduxjs/toolkit react-redux`
- 서버 상태: `@tanstack/react-query`, `@tanstack/react-query-devtools`
- HTTP: `axios`
- 유효성: `zod`, `react-hook-form`, `@hookform/resolvers`
- UI: `class-variance-authority`, `clsx`, `tailwind-merge`, `lucide-react`, `@radix-ui/react-*`

**devDependencies**:
- 빌드: `vite`, `@vitejs/plugin-react`
- TypeScript: `typescript`, `@types/react`, `@types/react-dom`, `@types/node`
- CSS: `tailwindcss`, `postcss`, `autoprefixer`
- 린트: `eslint`, `@typescript-eslint/*`, `eslint-plugin-react-hooks`, `prettier`, `eslint-config-prettier`
- e2e: `@playwright/test`

### 3단계: TypeScript 설정 (tsconfig.json)
- `strict: true` 필수
- `baseUrl: "."`, `paths: { "@/*": ["src/*"] }` — 절대경로 import
- `target: "ES2020"`, `lib: ["ES2020", "DOM", "DOM.Iterable"]`

### 4단계: ESLint + Prettier 설정
- `.eslintrc.cjs`: typescript-eslint + react-hooks 플러그인, `no-explicit-any: error`
- `.prettierrc`: `singleQuote: true`, `trailingComma: "all"`, `semi: false`, `printWidth: 100`
- `vite.config.ts` path alias와 동기화

### 5단계: Tailwind CSS + shadcn/ui 설정
- `tailwind.config.ts`: content 경로, `darkMode: "class"`, CSS 변수 기반 컬러 팔레트
- `postcss.config.js`: tailwindcss + autoprefixer
- `src/index.css`: Tailwind base/components/utilities + CSS 변수 정의

### 6단계: 디렉터리 구조 생성
```
src/
├── pages/          # 페이지 컴포넌트 (라우트 단위)
├── components/     # 공통·재사용 컴포넌트
│   └── ui/         # shadcn/ui 생성 컴포넌트
├── hooks/          # Custom Hooks
├── api/            # API 클라이언트 함수 (도메인별)
├── store/          # Zustand store (도메인별 slice)
├── types/          # TypeScript 타입 정의
├── utils/          # 순수 유틸리티 함수
├── lib/            # 라이브러리 초기화 (axios, queryClient 등)
├── App.tsx         # createBrowserRouter 기반 라우터
├── main.tsx        # QueryClientProvider + RouterProvider
└── index.css       # Tailwind 진입점
```

### 7단계: 핵심 설정 파일 생성
- `src/lib/axios.ts`: Axios 인스턴스, 요청·응답 인터셉터(토큰 주입, 401 처리)
- `src/lib/queryClient.ts`: QueryClient (staleTime 5분, gcTime 10분, retry 1)
- `src/utils/cn.ts`: `clsx + tailwind-merge` 조합 `cn()` 함수
- `src/App.tsx`: React Router v6 createBrowserRouter
- `src/main.tsx`: 앱 진입점 (QueryClientProvider + RouterProvider)

---

## 생성 원칙

- **경로 alias**: `@/` → `src/` 통일 (tsconfig + vite.config 동기화)
- **환경 변수**: `.env.example` 생성, `.env`는 `.gitignore` 처리
- **인라인 스타일 금지**: Tailwind 유틸리티 클래스 사용
- **cn() 함수**: 모든 조건부 클래스 조합에 `cn()` 사용

---

## 산출물

| 파일 | 설명 |
|:---|:---|
| `package.json` | 의존성·스크립트 전체 |
| `vite.config.ts` | 빌드·proxy·alias 설정 |
| `tsconfig.json` + `tsconfig.node.json` | TypeScript strict 설정 |
| `.eslintrc.cjs` | ESLint 규칙 |
| `.prettierrc` | Prettier 포맷 설정 |
| `tailwind.config.ts` | Tailwind 커스텀 설정 |
| `postcss.config.js` | PostCSS 플러그인 |
| `src/index.css` | Tailwind + CSS 변수 |
| `src/main.tsx` | 앱 진입점 |
| `src/App.tsx` | 라우터 루트 |
| `src/lib/axios.ts` | Axios 인스턴스 + 인터셉터 |
| `src/lib/queryClient.ts` | TanStack Query 클라이언트 |
| `src/utils/cn.ts` | cn() 유틸리티 |
| `.env.example` | 환경 변수 예시 |

template.md에서 각 파일의 기본 코드 골격을 참조한다.
