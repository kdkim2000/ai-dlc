# PLAN-007: AI-DLC 개발단계(프론트엔드-Next.js) 스킬 13종 일괄 생성

| 항목 | 내용 |
|:---|:---|
| 작성일 | 2026-05-23 |
| 상태 | 완료 |
| 전제 플랜 | PLAN-006 (ai-dlc-fe-* 스킬 18종) |
| 스킬 접두사 | `ai-dlc-nxt-*` |
| 스킬 경로 | `C:\Users\kdkim2000\.claude\skills\ai-dlc-nxt-*\` |

---

## Context

PLAN-006에서 React/Vite 기반 프론트엔드 스킬 18종(`ai-dlc-fe-*`)이 완성되었다. Next.js는 App Router·Server Components·Server Actions·Route Handlers 등 Vite 기반 React와 구조가 근본적으로 달라 별도 스킬 세트가 필요하다.

핵심 차이점:
- **App Router**: 파일시스템 라우팅(`page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`)
- **Server Components(RSC)**: 기본이 서버 컴포넌트, `'use client'` 명시 시에만 클라이언트 컴포넌트
- **Server Actions**: `'use server'` 지시어로 폼·뮤테이션을 서버에서 처리
- **Route Handlers**: `app/api/[route]/route.ts`에서 GET/POST 등 처리
- **데이터 패칭**: RSC에서 `fetch()` + `cache`/`revalidate` 옵션 활용
- **환경 변수**: `NEXT_PUBLIC_` 접두사 (Vite의 `VITE_` 대체)
- **배포**: Vercel 최적화, standalone Docker, ISR/SSG/SSR 전략

`ai-dlc-fe-*` 스킬 중 9종은 Next.js에서도 동일하게 재사용 가능하므로 새로 생성하지 않았다.

---

## 사용자 요구사항

> `@plans\PLAN-006_fe-react-skills-batch.md 을 참고하여 Next.js 기반 개발에 필요한 skills 를 추가할 계획을 수립하고 실행한다.`

---

## 설계 결정 사항

| 항목 | 결정 | 근거 |
|:---|:---|:---|
| 총 스킬 수 | 13종 | Next.js 고유 기능 커버 + fe-* 9종 재사용 |
| 스킬 접두사 | `ai-dlc-nxt-*` | nxt = Next.js; fe-* 와 명확 구분 |
| Next.js 버전 | 15 (App Router) | 최신 안정 버전; Pages Router 미지원 |
| RSC/CC 경계 | SKILL.md에 판단 기준 내장 | 'use client' 과다 사용이 가장 흔한 실수 |
| 데이터 패칭 전략 | RSC fetch() 우선, CC는 React Query 허용 | 서버 상태는 RSC에서, 클라이언트 인터랙션은 React Query |
| 인증 | Auth.js v5 (NextAuth.js) | Next.js App Router 공식 권장 라이브러리 |
| allowed-tools 분류 | 2-tier: create/revise=Read Grep Glob Write Edit, validate/guide=Read Grep Glob | PLAN-005~006 동일 패턴 |
| template.md 보유 스킬 | 6종 (create 그룹 전체) | validate/revise/guide는 콘텐츠가 SKILL.md에 내장 |
| ai-dlc-fe-* 재사용 스킬 | 9종 (새로 생성 안 함) | 동일 내용이므로 중복 생성 불필요 |

---

## ai-dlc-fe-* 재사용 스킬 (새로 생성하지 않음)

| 스킬명 | 재사용 이유 |
|:---|:---|
| `ai-dlc-fe-ts-check` | TypeScript 검사는 프레임워크 무관 |
| `ai-dlc-fe-lint-check` | ESLint 규칙 Next.js에서도 동일 적용 |
| `ai-dlc-fe-shadcn-guide` | shadcn/ui는 Next.js에서도 동일하게 동작 |
| `ai-dlc-fe-tailwind-guide` | Tailwind CSS는 프레임워크 무관 |
| `ai-dlc-fe-zod-guide` | Zod 스키마 검증은 프레임워크 무관 |
| `ai-dlc-fe-state-guide` | Zustand 전역 상태 패턴 동일 |
| `ai-dlc-fe-react-query-guide` | Client Components에서 React Query 패턴 동일 |
| `ai-dlc-fe-e2e-test-validate` | EV-001~007 검증 기준 동일 |
| `ai-dlc-fe-e2e-test-revise` | 테스트 수정 패턴 동일 |

---

## 스킬 목록 (13종)

| # | 스킬명 | 역할 | 유형 |
|:---:|:---|:---|:---:|
| 1 | `ai-dlc-nxt-project-setup` | Next.js 15 App Router 프로젝트 초기 설정 생성 | create |
| 2 | `ai-dlc-nxt-impl-plan` | Next.js 구현 전략 계획 문서 생성 | create |
| 3 | `ai-dlc-nxt-page-gen` | 페이지·레이아웃·서버/클라이언트 컴포넌트 코드 생성 | create |
| 4 | `ai-dlc-nxt-route-handler-gen` | Route Handlers(`app/api/`) 코드 생성 | create |
| 5 | `ai-dlc-nxt-server-action-gen` | Server Actions(`'use server'`) 코드 생성 | create |
| 6 | `ai-dlc-nxt-e2e-test-gen` | Playwright e2e 테스트 코드 생성 (Next.js App Router용) | create |
| 7 | `ai-dlc-nxt-code-review` | Next.js 코드 품질 검토 (RSC 경계·'use client' 과다 등) | validate |
| 8 | `ai-dlc-nxt-code-revise` | Next.js 코드 리뷰 결과 반영 | revise |
| 9 | `ai-dlc-nxt-sc-guide` | Server Components 데이터 패칭·캐싱·스트리밍 가이드 | guide |
| 10 | `ai-dlc-nxt-auth-guide` | 인증 가이드 (Auth.js v5 / NextAuth.js) | guide |
| 11 | `ai-dlc-nxt-middleware-guide` | Next.js 미들웨어 가이드 (인증·리다이렉트·i18n) | guide |
| 12 | `ai-dlc-nxt-perf-guide` | 성능 최적화 가이드 (next/image·next/font·PPR) | guide |
| 13 | `ai-dlc-nxt-deploy-guide` | 배포 가이드 (Vercel·standalone Docker·env vars) | guide |

---

## 생성된 파일 구조

```
C:\Users\kdkim2000\.claude\skills\
├── ai-dlc-nxt-project-setup\      SKILL.md + template.md  ✅
├── ai-dlc-nxt-impl-plan\          SKILL.md + template.md  ✅
├── ai-dlc-nxt-page-gen\           SKILL.md + template.md  ✅
├── ai-dlc-nxt-route-handler-gen\  SKILL.md + template.md  ✅
├── ai-dlc-nxt-server-action-gen\  SKILL.md + template.md  ✅
├── ai-dlc-nxt-e2e-test-gen\       SKILL.md + template.md  ✅
├── ai-dlc-nxt-code-review\        SKILL.md                ✅
├── ai-dlc-nxt-code-revise\        SKILL.md                ✅
├── ai-dlc-nxt-sc-guide\           SKILL.md                ✅
├── ai-dlc-nxt-auth-guide\         SKILL.md                ✅
├── ai-dlc-nxt-middleware-guide\   SKILL.md                ✅
├── ai-dlc-nxt-perf-guide\         SKILL.md                ✅
└── ai-dlc-nxt-deploy-guide\       SKILL.md                ✅
```

**총 파일: SKILL.md 13개 + template.md 6개 = 19개**

---

## 스킬별 핵심 설계

### ai-dlc-nxt-project-setup
- **트리거**: "Next.js 프로젝트 만들어줘", "create-next-app 설정"
- **생성 파일**: `package.json`(Next.js 15 + Auth.js v5 + TanStack Query v5 + Zustand + Prisma), `next.config.ts`, `tsconfig.json`(strict), `.eslintrc.cjs`, `tailwind.config.ts`, `src/lib/auth.ts`(CredentialsProvider + JWT/Session 콜백), `src/middleware.ts`, `.env.example`
- **디렉터리**: `src/app/`, `src/components/ui/`, `src/lib/`, `src/types/`, `src/actions/`

### ai-dlc-nxt-impl-plan
- **트리거**: "Next.js 구현 계획 세워줘", "App Router 구현 전략"
- **절차**: SCR-NNN → 라우트 매핑 → RSC/CC 경계 결정 → 데이터 패칭 전략 → Server Actions vs Route Handlers 선택
- **산출물**: `Next.js구현계획_{YYYYMMDD}.md` (라우트 트리 + RSC/CC 표 + 데이터 패칭 표 + 스프린트 계획)

### ai-dlc-nxt-page-gen
- **트리거**: "Next.js 페이지 만들어줘", "서버 컴포넌트 코드 생성"
- **원칙**: RSC 기본; CC는 인터랙션 필요 시에만; `next/image`·`next/link` 필수; 파일당 150줄 한도
- **생성 파일**: `page.tsx`(RSC + Suspense), `layout.tsx`, `loading.tsx`(스켈레톤), `error.tsx`(CC 에러 경계), `components/[Domain]Table.tsx`(RSC), `components/[Domain]Form.tsx`(CC)

### ai-dlc-nxt-route-handler-gen
- **트리거**: "API 라우트 만들어줘", "Route Handler 생성"
- **패턴**: `auth()` 인증 체크 → 요청 파싱 → Zod 검증 → 비즈니스 로직 → `NextResponse.json(ApiResponse<T>)`
- **생성 파일**: `app/api/[domain]/route.ts`(GET/POST), `app/api/[domain]/[id]/route.ts`(GET/PUT/DELETE), `src/types/api.ts`

### ai-dlc-nxt-server-action-gen
- **트리거**: "Server Action 만들어줘", "폼 서버 처리 코드"
- **패턴**: `'use server'` → `auth()` 인증 → Zod `safeParse` → DB 처리 → `revalidatePath`/`redirect`
- **FormState 타입**: `{ error: string | null; fieldErrors?: Record<string, string[]> }`

### ai-dlc-nxt-e2e-test-gen
- **트리거**: "Next.js e2e 테스트 만들어줘", "App Router Playwright 테스트"
- **특화**: `webServer: next dev`, 인증 픽스처(`authenticatedPage`), Server Action 폼 제출 + `waitForURL`, `data-testid` 선택자

### ai-dlc-nxt-code-review
- **이슈 코드**: NX-001~010 (Next.js 고유) + TC/SC/PF/A11Y (fe-* 재사용)
- **높음 이슈**: NX-001('use client' 과다), NX-002(CC서버로직), NX-005(인증누락), NX-006(Zod누락), NX-008(env노출)

### ai-dlc-nxt-code-revise
- **수정 우선순위**: SC-001 > NX-002/005/006/008 > TC-001/003 > NX-001 > NX-003/004 > NX-007/009 > A11Y > NX-010

### 가이드 스킬 (sc-guide, auth-guide, middleware-guide, perf-guide, deploy-guide)
- Auth.js v5 `auth()` / `useSession()` 이중 접근 패턴
- `middleware.ts` Edge Runtime 제약사항 및 matcher 설정
- `next/image` fill+sizes 패턴, `next/font` 최적화
- Standalone Docker (`output: 'standalone'`) + Dockerfile + docker-compose.yml
- ISR 전략 (force-cache / revalidate:N / no-store / revalidateTag)

---

## 검증 방법

| 트리거 문장 | 기대 스킬 |
|:---|:---|
| "Next.js 프로젝트 만들어줘" | `ai-dlc-nxt-project-setup` |
| "App Router 구현 계획 세워줘" | `ai-dlc-nxt-impl-plan` |
| "사용자 목록 페이지 만들어줘" | `ai-dlc-nxt-page-gen` |
| "GET /api/users Route Handler 만들어줘" | `ai-dlc-nxt-route-handler-gen` |
| "사용자 등록 Server Action 만들어줘" | `ai-dlc-nxt-server-action-gen` |
| "Next.js Playwright 테스트 생성" | `ai-dlc-nxt-e2e-test-gen` |
| "Next.js 코드 리뷰해줘" | `ai-dlc-nxt-code-review` |
| "RSC 리뷰 결과 코드에 반영해줘" | `ai-dlc-nxt-code-revise` |
| "Server Component 데이터 패칭 방법" | `ai-dlc-nxt-sc-guide` |
| "Auth.js로 로그인 구현 방법" | `ai-dlc-nxt-auth-guide` |
| "Next.js middleware.ts 설정법" | `ai-dlc-nxt-middleware-guide` |
| "next/image 사용법" | `ai-dlc-nxt-perf-guide` |
| "Vercel 배포 방법" | `ai-dlc-nxt-deploy-guide` |

---

## 비범위

- Pages Router 기반 Next.js (별도 PLAN 고려)
- React Native / Expo (모바일)
- GraphQL / tRPC 연동
- Prisma ORM 생성 스킬 (별도 PLAN 고려)
- Storybook 컴포넌트 문서화
- CI/CD 파이프라인 자동화 스킬 (GitHub Actions 예시는 deploy-guide에 포함)
- 단위 테스트(Vitest/Jest)
