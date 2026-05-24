# 프론트엔드 구현 계획서 템플릿

산출물 파일명: `프론트엔드구현계획_{사업명}_{YYYYMMDD}.md`

---

## 프론트엔드 구현 계획서

| 항목 | 내용 |
|:---|:---|
| 사업명 | {{사업명}} |
| 작성일 | {{YYYY-MM-DD}} |
| 작성자 | {{작성자}} |
| 버전 | v1.0 |
| 전제 산출물 | 화면설계서 v{{N}}, API설계서 v{{N}} |

---

## 1. 기술 스택

| 구분 | 기술 | 버전 | 비고 |
|:---|:---|:---|:---|
| 프레임워크 | React | 18.x | |
| 빌드 도구 | Vite | 5.x | |
| 언어 | TypeScript | 5.x (strict) | |
| 상태 관리 (클라이언트) | Zustand | 5.x | |
| 상태 관리 (서버) | TanStack Query | 5.x | |
| HTTP | Axios | 1.x | 인터셉터 공통 처리 |
| 유효성 검증 | Zod + React Hook Form | | |
| UI 컴포넌트 | shadcn/ui | | Radix UI 기반 |
| CSS | Tailwind CSS | 3.x | |
| 라우팅 | React Router | v6 | createBrowserRouter |
| e2e 테스트 | Playwright | 1.x | |

---

## 2. 화면 목록 및 우선순위

| SCR-ID | 화면명 | 화면 유형 | 연계 UC | 우선순위 | 구현 스프린트 |
|:---|:---|:---:|:---|:---:|:---:|
| SCR-001 | 로그인 | AUTH | UC-001 | High | S1 |
| SCR-002 | 사용자 목록 | LIST | UC-002 | High | S1 |
| SCR-003 | 사용자 등록 | FORM | UC-003 | High | S1 |
| SCR-004 | 사용자 수정 | FORM | UC-004 | Medium | S2 |
| SCR-005 | 사용자 상세 | DETAIL | UC-005 | Medium | S2 |
| SCR-006 | 대시보드 | DASHBOARD | UC-006 | Low | S3 |

> 화면 유형: LIST / FORM / DETAIL / DASHBOARD / AUTH / MODAL

---

## 3. 공통 컴포넌트 목록

| 컴포넌트명 | 파일 경로 | 사용 화면 | 의존성 |
|:---|:---|:---|:---|
| Layout | `components/Layout.tsx` | 전체 (인증 후) | Sidebar, Header |
| Sidebar | `components/Sidebar.tsx` | 전체 | — |
| Header | `components/Header.tsx` | 전체 | — |
| DataTable | `components/DataTable.tsx` | LIST 화면 | shadcn/ui Table |
| SearchForm | `components/SearchForm.tsx` | LIST 화면 | React Hook Form |
| FormField | `components/FormField.tsx` | FORM 화면 | shadcn/ui Label, Input |
| FormSelect | `components/FormSelect.tsx` | FORM 화면 | shadcn/ui Select |
| ConfirmDialog | `components/ConfirmDialog.tsx` | FORM/LIST | shadcn/ui Dialog |
| Spinner | `components/Spinner.tsx` | 전체 | — |
| Pagination | `components/Pagination.tsx` | LIST 화면 | — |

---

## 4. 화면별 구현 순서

### 1단계: 기반 작업 (공통)
- [ ] 공통 레이아웃 컴포넌트 (Layout, Sidebar, Header)
- [ ] shadcn/ui 기본 컴포넌트 설치 (Button, Input, Label, Select, Dialog, Table, Toast)
- [ ] DataTable, SearchForm, Pagination 공통 컴포넌트 구현
- [ ] 인증 스토어 (useAuthStore) 구현

### 2단계: 인증 화면
- [ ] SCR-001 로그인 페이지 (LoginPage)
  - useLoginMutation (Axios POST /api/auth/login)
  - Zod 스키마 + React Hook Form
  - 토큰 저장 (localStorage → useAuthStore)

### 3단계: 우선순위 High 화면
- [ ] SCR-002 사용자 목록 (UserListPage)
  - useUserList (useQuery)
  - DataTable + SearchForm + Pagination
- [ ] SCR-003 사용자 등록 (UserCreatePage)
  - useCreateUser (useMutation)
  - Zod 스키마 + 폼 유효성

### 4단계: 우선순위 Medium 화면
- [ ] SCR-004 사용자 수정 (UserEditPage)
- [ ] SCR-005 사용자 상세 (UserDetailPage)

### 5단계: 우선순위 Low 화면
- [ ] SCR-006 대시보드 (DashboardPage)

---

## 5. 상태 관리 전략

| 상태 유형 | 관리 방법 | 사용 예시 |
|:---|:---|:---|
| 서버 상태 (API 데이터) | TanStack Query useQuery | 사용자 목록, 상세 정보 |
| 뮤테이션 (CUD 작업) | TanStack Query useMutation | 사용자 등록, 수정, 삭제 |
| 인증 상태 (전역) | Zustand useAuthStore | 로그인 토큰, 사용자 프로필 |
| UI 상태 (전역) | Zustand useUIStore | 사이드바 열림/닫힘 |
| 폼 상태 (로컬) | React Hook Form | 등록·수정 폼 |
| 단순 UI 토글 | useState | 모달 열림, 탭 선택 |

### Zustand store 구조

```typescript
// src/store/useAuthStore.ts
interface AuthStore {
  user: User | null
  accessToken: string | null
  setAuth: (user: User, token: string) => void
  logout: () => void
}
```

### queryKey 설계

```typescript
// src/api/queryKeys.ts
export const userKeys = {
  all: ['users'] as const,
  list: (filters?: UserFilter) => [...userKeys.all, 'list', filters] as const,
  detail: (id: number) => [...userKeys.all, 'detail', id] as const,
}
```

---

## 6. API 연동 전략

### Axios 인스턴스 (`src/lib/axios.ts`)
- baseURL: `import.meta.env.VITE_API_URL`
- 요청 인터셉터: Bearer 토큰 자동 주입
- 응답 인터셉터: 401 → 로그인 리다이렉트

### API 함수 파일 구조

```
src/api/
├── user.api.ts       # 사용자 API 함수
├── auth.api.ts       # 인증 API 함수
└── ...
```

### API 함수 패턴

```typescript
// src/api/user.api.ts
import apiClient from '@/lib/axios'
import type { ApiResponse, UserVO, UserFilter } from '@/types'

export const userApi = {
  getList: (filter: UserFilter) =>
    apiClient.get<ApiResponse<UserVO[]>>('/users', { params: filter }).then((r) => r.data.data),
  getOne: (id: number) =>
    apiClient.get<ApiResponse<UserVO>>(`/users/${id}`).then((r) => r.data.data),
  create: (data: UserCreateReq) =>
    apiClient.post<ApiResponse<UserVO>>('/users', data).then((r) => r.data.data),
  update: (id: number, data: UserUpdateReq) =>
    apiClient.put<ApiResponse<UserVO>>(`/users/${id}`, data).then((r) => r.data.data),
  remove: (id: number) =>
    apiClient.delete(`/users/${id}`),
}
```

---

## 7. 파일 명명 규칙

| 대상 | 규칙 | 예시 |
|:---|:---|:---|
| 페이지 컴포넌트 | `{도메인명}{화면유형}Page.tsx` | `UserListPage.tsx` |
| 공통 컴포넌트 | `{컴포넌트명}.tsx` (PascalCase) | `DataTable.tsx` |
| Custom Hook | `use{도메인명}{동작}.ts` | `useUserList.ts` |
| API 함수 파일 | `{도메인명}.api.ts` | `user.api.ts` |
| Zustand store | `use{도메인명}Store.ts` | `useAuthStore.ts` |
| 타입 정의 | `{도메인명}.types.ts` | `user.types.ts` |
| queryKey 파일 | `queryKeys.ts` (통합) | `src/api/queryKeys.ts` |
