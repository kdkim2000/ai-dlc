# Next.js App Router 구현 계획서 템플릿

## Next.js구현계획_{YYYYMMDD}.md

```markdown
# Next.js App Router 구현 계획서

| 항목 | 내용 |
|:---|:---|
| 작성일 | YYYY-MM-DD |
| Next.js 버전 | 15 (App Router) |
| 기반 설계서 | PLAN-004 화면설계서, API설계서 |

---

## 기술 스택

| 카테고리 | 결정 | 비고 |
|:---|:---|:---|
| 프레임워크 | Next.js 15 App Router | - |
| 언어 | TypeScript 5.x strict | - |
| 인증 | Auth.js v5 | Credentials Provider |
| 스타일 | Tailwind CSS + shadcn/ui | - |
| 서버 상태 | RSC fetch() + react-cache | 기본 |
| 클라이언트 상태 | TanStack Query v5 | CC 내 API 연동 |
| 전역 상태 | Zustand | UI 상태 전용 |
| 폼 | React Hook Form + Zod | - |
| DB | Prisma + PostgreSQL | (선택) |

---

## 화면 목록 및 라우트 구조

| SCR-ID | 화면명 | App Router 경로 | 유형 | 우선순위 |
|:---|:---|:---|:---:|:---:|
| SCR-001 | 로그인 | `app/(auth)/login/page.tsx` | CC | High |
| SCR-002 | 대시보드 홈 | `app/(dashboard)/page.tsx` | RSC | High |
| SCR-003 | 사용자 목록 | `app/(dashboard)/users/page.tsx` | RSC | High |
| SCR-004 | 사용자 등록 | `app/(dashboard)/users/new/page.tsx` | CC | High |
| SCR-005 | 사용자 수정 | `app/(dashboard)/users/[id]/edit/page.tsx` | CC | Medium |
| SCR-006 | 사용자 상세 | `app/(dashboard)/users/[id]/page.tsx` | RSC | Medium |

---

## App Router 디렉터리 구조

```
src/app/
├── layout.tsx                     # Root Layout (전체 Providers)
├── page.tsx                       # 루트 홈 → /dashboard 리다이렉트
├── (auth)/                        # Route Group: 인증 불필요
│   └── login/
│       └── page.tsx
├── (dashboard)/                   # Route Group: 인증 필요
│   ├── layout.tsx                 # Dashboard Layout (사이드바, 헤더)
│   ├── page.tsx                   # 대시보드 홈
│   └── users/
│       ├── page.tsx               # 사용자 목록 (RSC)
│       ├── new/
│       │   └── page.tsx           # 사용자 등록 (CC + Server Action)
│       └── [id]/
│           ├── page.tsx           # 사용자 상세 (RSC)
│           └── edit/
│               └── page.tsx      # 사용자 수정 (CC + Server Action)
└── api/
    └── users/
        ├── route.ts               # GET(목록), POST(생성)
        └── [id]/
            └── route.ts           # GET(단건), PUT(수정), DELETE(삭제)
```

---

## RSC / CC 분류

| 화면/컴포넌트 | 분류 | 이유 |
|:---|:---:|:---|
| `users/page.tsx` | RSC | 데이터 조회만, 인터랙션 없음 |
| `users/new/page.tsx` | CC | React Hook Form, 이벤트 핸들러 |
| `users/[id]/page.tsx` | RSC | 데이터 조회만 |
| `components/UserTable.tsx` | RSC | 정적 테이블, 데이터 Props로 수신 |
| `components/UserForm.tsx` | CC | 폼 입력, 유효성 검사 |
| `components/DeleteButton.tsx` | CC | onClick 이벤트, 확인 Dialog |
| `components/SearchInput.tsx` | CC | useState, onChange |
| `app/(dashboard)/layout.tsx` | CC | 모바일 사이드바 토글 상태 |

---

## 데이터 패칭 전략

| 화면 | 전략 | 코드 패턴 |
|:---|:---|:---|
| 사용자 목록 (RSC) | fetch + revalidate | `fetch('/api/users', { next: { revalidate: 60 } })` |
| 사용자 상세 (RSC) | fetch + cache | `fetch('/api/users/1', { cache: 'force-cache' })` |
| 실시간 검색 (CC) | TanStack Query | `useQuery({ queryKey: ['users', keyword] })` |
| 등록/수정/삭제 | Server Actions | `'use server'` + `revalidatePath('/users')` |

---

## Server Actions 목록

| Action명 | 파일 | UC | 설명 |
|:---|:---|:---|:---|
| `createUser` | `actions/user.ts` | UC-003 | 사용자 등록 + 목록 캐시 무효화 |
| `updateUser` | `actions/user.ts` | UC-004 | 사용자 수정 + 상세·목록 캐시 무효화 |
| `deleteUser` | `actions/user.ts` | UC-005 | 사용자 삭제 + 목록 캐시 무효화 |

---

## Route Handlers 목록

| 경로 | 메서드 | UC | 설명 |
|:---|:---|:---|:---|
| `/api/users` | GET | UC-002 | 사용자 목록 조회 (페이지네이션) |
| `/api/users` | POST | UC-003 | 사용자 생성 |
| `/api/users/[id]` | GET | UC-006 | 사용자 단건 조회 |
| `/api/users/[id]` | PUT | UC-004 | 사용자 수정 |
| `/api/users/[id]` | DELETE | UC-005 | 사용자 삭제 |

---

## 구현 단계 (스프린트 계획)

### Phase 1: 기반 설정 (Sprint 1)
- [ ] `ai-dlc-nxt-project-setup` 실행
- [ ] Auth.js 인증 구현 (로그인/로그아웃)
- [ ] Root Layout + Dashboard Layout 구현
- [ ] 공통 컴포넌트 (DataTable, FormField, Modal, Toast)

### Phase 2: 핵심 CRUD (Sprint 2)
- [ ] 사용자 목록 (RSC + Route Handler GET)
- [ ] 사용자 등록 (CC + Server Action createUser)
- [ ] 사용자 수정 (CC + Server Action updateUser)
- [ ] 사용자 삭제 (CC + Server Action deleteUser)

### Phase 3: 검증 및 e2e (Sprint 3)
- [ ] `ai-dlc-nxt-code-review` 실행
- [ ] `ai-dlc-nxt-e2e-test-gen` 실행
- [ ] 코드 수정 및 테스트 보완
```
