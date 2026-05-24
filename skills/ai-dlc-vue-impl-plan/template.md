# Vue구현계획 템플릿

```markdown
# Vue.js 구현 계획서

| 항목 | 내용 |
|:---|:---|
| 사업명 | {{사업명}} |
| 작성일 | {{YYYY-MM-DD}} |
| 작성자 | {{작성자}} |
| Vue.js 버전 | 3.x (Composition API + `<script setup>`) |
| 빌드 도구 | Vite 5.x |

---

## 1. 화면 목록 및 우선순위

| 화면ID | 화면명 | 유형 | 연계 UC | 우선순위 | 비고 |
|:---:|:---|:---:|:---:|:---:|:---|
| SCR-001 | 로그인 | AUTH | UC-001 | High | |
| SCR-002 | 홈/대시보드 | DASHBOARD | UC-002 | High | |
| SCR-003 | 사용자 목록 | LIST | UC-003 | High | |
| SCR-004 | 사용자 등록 | FORM | UC-004 | High | |
| SCR-005 | 사용자 상세 | DETAIL | UC-005 | Medium | |
<!-- TODO: 화면설계서 기반으로 완성 -->

---

## 2. 라우트 구조

```
src/router/index.ts
├── /login                  → LoginView.vue           (requiresAuth: false)
├── / (AppLayout.vue)       [requiresAuth: true]
│   ├── /                   → HomeView.vue
│   ├── /users              → UsersView.vue
│   ├── /users/new          → UserCreateView.vue
│   ├── /users/:id          → UserDetailView.vue
│   └── /users/:id/edit     → UserEditView.vue
└── /:pathMatch(.*)         → NotFoundView.vue
```

| 경로 | View 컴포넌트 | 화면ID | 지연로딩 | 인증 필요 |
|:---|:---|:---:|:---:|:---:|
| `/login` | `LoginView.vue` | SCR-001 | O | X |
| `/` | `HomeView.vue` | SCR-002 | O | O |
| `/users` | `UsersView.vue` | SCR-003 | O | O |
| `/users/new` | `UserCreateView.vue` | SCR-004 | O | O |
| `/users/:id` | `UserDetailView.vue` | SCR-005 | O | O |
<!-- TODO: 화면설계서 기반으로 완성 -->

---

## 3. 컴포넌트 분류

### 3.1 View 컴포넌트 (라우트 단위)

| 파일 | 화면ID | 데이터 패칭 | 사용 Composable |
|:---|:---:|:---|:---|
| `src/views/LoginView.vue` | SCR-001 | - | `useAuth` |
| `src/views/HomeView.vue` | SCR-002 | useQuery(dashboard) | `useDashboard` |
| `src/views/UsersView.vue` | SCR-003 | useQuery(userList) | `useUserList` |
| `src/views/UserCreateView.vue` | SCR-004 | - | `useUserForm` |
| `src/views/UserDetailView.vue` | SCR-005 | useQuery(userById) | `useUserDetail` |

### 3.2 공통 컴포넌트

| 컴포넌트 | 파일 | 역할 | 사용 화면 |
|:---|:---|:---|:---|
| AppLayout | `components/AppLayout.vue` | 레이아웃(Sidebar + Header + RouterView) | 인증 필요 전체 |
| Sidebar | `components/Sidebar.vue` | 메뉴 네비게이션 | AppLayout |
| AppHeader | `components/AppHeader.vue` | 헤더(사용자 정보, 로그아웃) | AppLayout |
| DataTable | `components/DataTable.vue` | 데이터 테이블(정렬·페이지네이션) | LIST 화면 전체 |
| SearchForm | `components/SearchForm.vue` | 검색 폼 | LIST 화면 전체 |
| ConfirmDialog | `components/ConfirmDialog.vue` | 삭제 확인 다이얼로그 | FORM/DETAIL 화면 |
| LoadingSpinner | `components/LoadingSpinner.vue` | 로딩 상태 표시 | 전체 |
| ErrorMessage | `components/ErrorMessage.vue` | 에러 메시지 표시 | 전체 |

### 3.3 도메인 컴포넌트 (예: 사용자 도메인)

| 컴포넌트 | 파일 | 계층 | 역할 |
|:---|:---|:---:|:---|
| UserTable | `components/UserTable.vue` | Presentational | 사용자 목록 표시 |
| UserForm | `components/UserForm.vue` | Presentational | 사용자 등록·수정 폼 |
| UserCard | `components/UserCard.vue` | Presentational | 사용자 상세 카드 |
<!-- TODO: 도메인별로 추가 -->

---

## 4. Pinia 스토어 목록

| 스토어 파일 | 스토어 ID | 상태 | Actions | 영속성 |
|:---|:---:|:---|:---|:---:|
| `stores/auth.ts` | `auth` | token, user | login, logout | O (token, user) |
| `stores/ui.ts` | `ui` | sidebarOpen, theme | toggleSidebar, setTheme | O (theme) |
<!-- TODO: 도메인별 UI 상태 스토어 추가 (서버 데이터는 Vue Query 담당) -->

> **원칙**: 서버 데이터(API 응답) → Vue Query, UI 상태 → Pinia, 컴포넌트 로컬 상태 → ref/reactive

---

## 5. Composable 목록

| Composable | 파일 | 역할 | 사용 View |
|:---|:---|:---|:---|
| `useAuth` | `composables/useAuth.ts` | 로그인·로그아웃·인증 상태 | LoginView |
| `useUserList` | `composables/useUserList.ts` | 사용자 목록 조회 + 필터 + 페이지네이션 | UsersView |
| `useUserForm` | `composables/useUserForm.ts` | 사용자 등록·수정 폼 + 저장 mutation | UserCreate/EditView |
| `useUserDetail` | `composables/useUserDetail.ts` | 사용자 단건 조회 + 삭제 mutation | UserDetailView |
| `useToast` | `composables/useToast.ts` | 알림 메시지 공통 처리 | 전체 |
<!-- TODO: 도메인별로 추가 -->

---

## 6. API 모듈 목록

| 파일 | 대상 도메인 | 함수 목록 |
|:---|:---|:---|
| `api/user.api.ts` | 사용자 | fetchUserList, fetchUserById, createUser, updateUser, deleteUser |
| `api/auth.api.ts` | 인증 | login, logout, refreshToken |
<!-- TODO: 도메인별로 추가 -->

### Vue Query queryKey 설계

```typescript
// 사용자 도메인 queryKey 예시
const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters: UserFilters) => [...userKeys.lists(), filters] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: number) => [...userKeys.details(), id] as const,
}
```

---

## 7. 구현 순서 (단계별)

### Phase 1: 프로젝트 기반 (1~2일)
- [ ] `ai-dlc-vue-project-setup` 실행 → 프로젝트 초기화
- [ ] AppLayout, Sidebar, AppHeader 공통 컴포넌트 구현
- [ ] 공통 UI 컴포넌트 (DataTable, SearchForm, ConfirmDialog)
- [ ] Pinia auth 스토어 + LoginView 구현

### Phase 2: 핵심 화면 (High 우선순위)
<!-- TODO: SCR-NNN 목록 기반으로 작업 목록 작성 -->
- [ ] SCR-003: 사용자 목록 (UsersView + UserTable + useUserList)
- [ ] SCR-004: 사용자 등록 (UserCreateView + UserForm + useUserForm)
- [ ] SCR-005: 사용자 상세·수정 (UserDetailView + useUserDetail)

### Phase 3: 보완 화면 (Medium/Low 우선순위)
<!-- TODO: Medium/Low SCR 목록 추가 -->

### Phase 4: 품질 검증
- [ ] `ai-dlc-vue-code-review` — 코드 품질 검토 (VV-001~010)
- [ ] `ai-dlc-vue-ts-check` — vue-tsc TypeScript 검사
- [ ] `ai-dlc-vue-lint-check` — ESLint 검사
- [ ] `ai-dlc-vue-code-revise` — 리뷰 결과 반영

### Phase 5: 테스트
- [ ] `ai-dlc-vue-e2e-test-gen` — Playwright e2e 테스트 생성
- [ ] `ai-dlc-fe-e2e-test-validate` — 테스트 검증
- [ ] `ai-dlc-fe-e2e-test-revise` — 테스트 수정

---

## 8. 기술 스택 및 파일 명명 규칙

### 기술 스택

| 분류 | 도구 | 버전 |
|:---|:---|:---|
| 프레임워크 | Vue | 3.x |
| 빌드 | Vite | 5.x |
| 언어 | TypeScript | 5.x |
| 상태관리 | Pinia | 2.x |
| 라우터 | Vue Router | 4.x |
| 서버 상태 | @tanstack/vue-query | 5.x |
| HTTP | Axios | 1.x |
| 폼 검증 | VeeValidate v4 + Zod | 4.x / 3.x |
| UI | shadcn-vue + Tailwind CSS | - |
| e2e 테스트 | Playwright | 1.x |

### 파일 명명 규칙

| 유형 | 규칙 | 예시 |
|:---|:---|:---|
| View 컴포넌트 | PascalCase + View 접미사 | `UsersView.vue` |
| 일반 컴포넌트 | PascalCase | `UserTable.vue` |
| Composable | camelCase + use 접두사 | `useUserList.ts` |
| Pinia 스토어 | camelCase | `userStore.ts` |
| API 모듈 | camelCase + .api 접미사 | `user.api.ts` |
| 타입 파일 | camelCase | `user.types.ts` |

---

## 문서 버전 이력

| 버전 | 일자 | 작성자 | 변경 내용 |
|:---|:---|:---|:---|
| v0.1 | {{YYYY-MM-DD}} | 초안 작성 | 최초 생성 |
```
