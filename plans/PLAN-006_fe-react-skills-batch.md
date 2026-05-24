# PLAN-006: AI-DLC 개발단계(프론트엔드-React) 스킬 18종 일괄 생성

## 메타

| 항목 | 내용 |
|:---|:---|
| 작성일 | 2026-05-23 |
| 상태 | 완료 |
| 전제 플랜 | PLAN-005 (백엔드-Spring Boot) |
| 스킬 경로 | `C:\Users\kdkim2000\.claude\skills\ai-dlc-fe-*` |

---

## Context

PLAN-001~005 완료로 요구사항·분석·코드분석·설계·백엔드 개발 단계 스킬(총 60종)이 갖춰졌다. 이번 계획은 **개발단계(Frontend-React)**에 필요한 18종 스킬을 일괄 생성한다. 화면설계서(SCR-NNN)·API설계서(operationId)를 React/TypeScript 실제 소스코드로 변환하는 단계다.

백엔드 스킬 접두사 `ai-dlc-sb-*`와 대칭되는 `ai-dlc-fe-*` (fe = Frontend)를 사용한다.

---

## 사용자 요구사항

AI-DLC 개발(프론트엔드-React)단계에서 필요한 skills 목록을 도출하고 일괄 생성:
- React/Vite 프로젝트 초기화
- Node.js/Express 프로젝트 초기화
- 프론트엔드 구현 전략 계획
- 계획에 따른 컴포넌트 구현
- 생성된 코드 품질 검토
- 리뷰 결과 반영
- TypeScript 타입 검사
- ESLint 코드 스타일 검사
- shadcn/ui 컴포넌트 가이드
- Tailwind CSS 가이드
- Axios HTTP 클라이언트 가이드
- Zod 스키마 검증 가이드
- React 성능 최적화 가이드
- Playwright 기반 e2e 테스트 코드 생성
- 그외 누락된 필수 스킬 추가 (4종 제안)

---

## 설계 결정 사항

| 항목 | 결정 | 근거 |
|:---|:---|:---|
| 총 스킬 수 | 18종 | 사용자 지정 14종 + 제안 4종 (e2e-test-validate, e2e-test-revise, state-guide, react-query-guide) |
| 스킬 접두사 | `ai-dlc-fe-*` | fe = Frontend; Vue·Svelte 등 확장 시에도 재사용 가능 |
| allowed-tools 분류 | 2-tier: create/revise=RGGwWE, validate/guide=RGG | 백엔드 스킬(PLAN-005)과 동일 패턴 |
| Node.js/Express 목적 | BFF 또는 목업 API 서버 | React 앱 개발 시 백엔드 대체 또는 프록시 용도 |
| 상태 관리 기본 라이브러리 | Zustand | 경량·간단한 API; Redux Toolkit 병기 |
| 서버 상태 관리 | TanStack Query (React Query v5) | Axios 연동, 캐싱·무효화 패턴 제공 |
| e2e 3종 세트 | gen/validate/revise | 백엔드 unit-test 세트와 대칭 |
| template.md 보유 스킬 | 5종 (create 그룹만) | validate/revise/guide는 콘텐츠가 SKILL.md에 내장 |
| TypeScript 버전 | 5.x strict mode | Vite + shadcn/ui 기본 권장 설정 |

---

## 생성된 파일 구조

```
C:\Users\kdkim2000\.claude\skills\
├── ai-dlc-fe-project-setup\      SKILL.md + template.md  ✅
├── ai-dlc-fe-node-setup\         SKILL.md + template.md  ✅
├── ai-dlc-fe-impl-plan\          SKILL.md + template.md  ✅
├── ai-dlc-fe-component-gen\      SKILL.md + template.md  ✅
├── ai-dlc-fe-e2e-test-gen\       SKILL.md + template.md  ✅
├── ai-dlc-fe-code-review\        SKILL.md                ✅
├── ai-dlc-fe-ts-check\           SKILL.md                ✅
├── ai-dlc-fe-lint-check\         SKILL.md                ✅
├── ai-dlc-fe-e2e-test-validate\  SKILL.md                ✅
├── ai-dlc-fe-code-revise\        SKILL.md                ✅
├── ai-dlc-fe-e2e-test-revise\    SKILL.md                ✅
├── ai-dlc-fe-shadcn-guide\       SKILL.md                ✅
├── ai-dlc-fe-tailwind-guide\     SKILL.md                ✅
├── ai-dlc-fe-axios-guide\        SKILL.md                ✅
├── ai-dlc-fe-zod-guide\          SKILL.md                ✅
├── ai-dlc-fe-perf-guide\         SKILL.md                ✅
├── ai-dlc-fe-state-guide\        SKILL.md                ✅
└── ai-dlc-fe-react-query-guide\  SKILL.md                ✅
```

**총 파일: SKILL.md 18개 + template.md 5개 = 23개**

---

## 스킬 목록 (18종)

| # | 스킬명 | 역할 | 유형 | allowed-tools |
|:---:|:---|:---|:---:|:---|
| 1 | `ai-dlc-fe-project-setup` | React/Vite 프로젝트 초기 설정 생성 | create | Read Grep Glob Write Edit |
| 2 | `ai-dlc-fe-node-setup` | Node.js/Express BFF·Mock 서버 초기 설정 생성 | create | Read Grep Glob Write Edit |
| 3 | `ai-dlc-fe-impl-plan` | 프론트엔드 구현 전략 계획 문서 생성 | create | Read Grep Glob Write Edit |
| 4 | `ai-dlc-fe-component-gen` | React 컴포넌트·Hook·API 클라이언트 코드 생성 | create | Read Grep Glob Write Edit |
| 5 | `ai-dlc-fe-e2e-test-gen` | Playwright 기반 e2e 테스트 코드 생성 | create | Read Grep Glob Write Edit |
| 6 | `ai-dlc-fe-code-review` | 생성된 프론트엔드 코드 품질 검토 | validate | Read Grep Glob |
| 7 | `ai-dlc-fe-ts-check` | TypeScript 타입 안전성 검사 | validate | Read Grep Glob |
| 8 | `ai-dlc-fe-lint-check` | ESLint 코드 스타일 검사 | validate | Read Grep Glob |
| 9 | `ai-dlc-fe-e2e-test-validate` | Playwright e2e 테스트 코드 품질 검토 | validate | Read Grep Glob |
| 10 | `ai-dlc-fe-code-revise` | 코드 리뷰 결과 반영 (소스코드 수정) | revise | Read Grep Glob Write Edit |
| 11 | `ai-dlc-fe-e2e-test-revise` | e2e 테스트 검증 결과 반영 (테스트 수정) | revise | Read Grep Glob Write Edit |
| 12 | `ai-dlc-fe-shadcn-guide` | shadcn/ui 컴포넌트 활용 가이드 | guide | Read Grep Glob |
| 13 | `ai-dlc-fe-tailwind-guide` | Tailwind CSS 활용 가이드 | guide | Read Grep Glob |
| 14 | `ai-dlc-fe-axios-guide` | Axios HTTP 클라이언트 활용 가이드 | guide | Read Grep Glob |
| 15 | `ai-dlc-fe-zod-guide` | Zod 스키마 검증 활용 가이드 | guide | Read Grep Glob |
| 16 | `ai-dlc-fe-perf-guide` | React 성능 최적화 가이드 | guide | Read Grep Glob |
| 17 | `ai-dlc-fe-state-guide` | 상태 관리 가이드 (Zustand / Redux Toolkit) | guide | Read Grep Glob |
| 18 | `ai-dlc-fe-react-query-guide` | TanStack Query(React Query) 서버 상태 관리 가이드 | guide | Read Grep Glob |

---

## 스킬별 핵심 설계

### 그룹 A: 프로젝트 초기 설정 (2종)

**`ai-dlc-fe-project-setup`**
- 트리거: "React 프로젝트 만들어줘", "Vite 프로젝트 초기화"
- 절차: React+TypeScript 초기화 → 의존성(Zustand, TanStack Query, Axios, Zod, shadcn/ui, Tailwind) → tsconfig strict → ESLint/Prettier → 디렉터리 구조 생성 → 핵심 파일(axios 인스턴스, queryClient, cn 유틸)
- template.md: package.json 전체, vite.config.ts, tsconfig.json, ESLint, Prettier, Tailwind, src/lib/ 파일들

**`ai-dlc-fe-node-setup`**
- 트리거: "Node.js 서버 만들어줘", "Express BFF 서버 초기화"
- 절차: Express + TypeScript 설정 → 미들웨어(cors, morgan, helmet) → 라우터 구조 → ApiResponse<T> 공용 타입
- template.md: package.json, app.ts, routes/, types/api-response.ts, errorHandler middleware

### 그룹 B: 구현 계획 및 코드 생성 (3종)

**`ai-dlc-fe-impl-plan`**
- 트리거: "프론트엔드 구현 계획 세워줘", "컴포넌트 구현 전략"
- 절차: 화면 목록 정리 → 공통 컴포넌트 추출 → 구현 순서 결정 → 상태/API 전략
- template.md: 구현계획서 마크다운 템플릿 (화면 목록, 단계별 구현, queryKey 설계, API 함수 모듈 패턴)

**`ai-dlc-fe-component-gen`**
- 트리거: "컴포넌트 만들어줘", "화면 구현해줘", "React 컴포넌트 생성"
- 절차: 라우팅 → 페이지 컴포넌트(최대 150줄) → 공통 컴포넌트(shadcn/ui) → Custom Hook(useQuery/useMutation) → API 클라이언트(Axios+타입) → Zod 스키마(폼 유효성)
- template.md: UserVO/ApiResponse 타입, Zod 스키마, API 함수, queryKeys, useUserList/useCreateUser 훅, UserListPage/UserCreatePage, DataTable 컴포넌트

**`ai-dlc-fe-e2e-test-gen`**
- 트리거: "e2e 테스트 만들어줘", "Playwright 테스트 생성"
- 절차: UC별 시나리오 도출 → Page Object Model 구성 → Happy Path + 경계값 + 에러 시나리오
- template.md: playwright.config.ts, BasePage/UserListPage/UserCreatePage, *.spec.ts, auth.fixture.ts

### 그룹 C: 코드 품질 검증 (4종)

**`ai-dlc-fe-code-review`** — TC(5)/LC(5)/PF(4)/SC(4)/A11Y(3) 검증, 심각도별 VI-NNN 이슈 목록

**`ai-dlc-fe-ts-check`** — TC-001(any) ~ TC-007(React 이벤트 타입)

**`ai-dlc-fe-lint-check`** — LN-001(exhaustive-deps) ~ LN-008(react-refresh)

**`ai-dlc-fe-e2e-test-validate`** — EV-001(UC커버리지) ~ EV-007(playwright.config 설정)

### 그룹 D: 코드 수정 (2종)

**`ai-dlc-fe-code-revise`** — 검토 보고서 이슈 → 수정 우선순위(SC>TC>PF>LC>A11Y>LN) → 최소 변경 적용

**`ai-dlc-fe-e2e-test-revise`** — e2e 검증 보고서 이슈 → 누락 시나리오 추가 / 선택자 수정 / 비동기 처리 보완

### 그룹 E: 개발 가이드라인 (7종)

| 스킬명 | 핵심 내용 |
|:---|:---|
| `ai-dlc-fe-shadcn-guide` | 설치, Button/Dialog/Form/Table/Select/Toast 사용법, cn() 유틸 |
| `ai-dlc-fe-tailwind-guide` | 반응형(sm/md/lg), flex/grid 패턴, @apply, tailwind.config.ts |
| `ai-dlc-fe-axios-guide` | 인스턴스 설정, 인터셉터(토큰주입/401처리), AxiosError 타입가드, React Query 연동 |
| `ai-dlc-fe-zod-guide` | 스키마 정의, zodResolver, superRefine, z.infer 타입 추출 |
| `ai-dlc-fe-perf-guide` | React.memo/useMemo/useCallback 기준, Code Splitting, 번들분석 |
| `ai-dlc-fe-state-guide` | 클라이언트 vs 서버 상태 구분, Zustand store, devtools+persist+immer |
| `ai-dlc-fe-react-query-guide` | QueryClient 설정, queryKey 설계, useMutation+invalidateQueries, Optimistic Update |

---

## 검증 방법

| 트리거 문장 | 기대 스킬 |
|:---|:---|
| "React/Vite 프로젝트 만들어줘" | `ai-dlc-fe-project-setup` |
| "Express BFF 서버 초기화해줘" | `ai-dlc-fe-node-setup` |
| "프론트엔드 구현 전략 세워줘" | `ai-dlc-fe-impl-plan` |
| "로그인 화면 컴포넌트 만들어줘" | `ai-dlc-fe-component-gen` |
| "Playwright 테스트 만들어줘" | `ai-dlc-fe-e2e-test-gen` |
| "생성된 컴포넌트 코드 리뷰해줘" | `ai-dlc-fe-code-review` |
| "TypeScript 타입 오류 찾아줘" | `ai-dlc-fe-ts-check` |
| "ESLint 코드 스타일 검사해줘" | `ai-dlc-fe-lint-check` |
| "e2e 테스트 코드 검토해줘" | `ai-dlc-fe-e2e-test-validate` |
| "리뷰 결과 코드에 반영해줘" | `ai-dlc-fe-code-revise` |
| "e2e 테스트 수정해줘" | `ai-dlc-fe-e2e-test-revise` |
| "shadcn/ui 컴포넌트 어떻게 써?" | `ai-dlc-fe-shadcn-guide` |
| "Tailwind CSS 가이드 알려줘" | `ai-dlc-fe-tailwind-guide` |
| "Axios 인터셉터 어떻게 설정해?" | `ai-dlc-fe-axios-guide` |
| "Zod로 폼 유효성 검사 하는 법" | `ai-dlc-fe-zod-guide` |
| "React 리렌더링 최적화 방법은?" | `ai-dlc-fe-perf-guide` |
| "Zustand 스토어 어떻게 만들어?" | `ai-dlc-fe-state-guide` |
| "React Query useQuery 사용법" | `ai-dlc-fe-react-query-guide` |

---

## 비범위

- 백엔드(Spring Boot) 코드 생성 (PLAN-005)
- CI/CD 파이프라인 설정 (GitHub Actions, Jenkins)
- 컨테이너(Docker/Kubernetes) 설정
- Storybook 컴포넌트 문서화
- 모바일(React Native) 개발
- 마이크로 프론트엔드(Module Federation) 설정
- Vue/Svelte 등 타 프레임워크 (별도 PLAN)
- 단위 테스트(Vitest/Jest) 생성 — e2e(Playwright)와 분리, 추후 PLAN-007 고려
