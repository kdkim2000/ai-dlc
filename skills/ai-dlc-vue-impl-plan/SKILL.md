---
name: ai-dlc-vue-impl-plan
description: AI-DLC 개발단계(프론트엔드-Vue.js) 스킬. 화면설계서와 API설계서를 기반으로 Vue.js 구현 전략 계획 문서를 생성한다. "Vue 구현 계획 세워줘", "Vue 화면 구현 순서", "Vue 컴포넌트 설계", "Vue 개발 전략", "Vue 구현 전략 문서", "뷰 개발 계획 수립", "Vue 화면 개발 계획" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Vue.js 구현 전략 계획

화면설계서(SCR-NNN)·API설계서·프로젝트 구조를 분석하여 View/컴포넌트 구현 순서·Pinia 스토어 목록·Composable 목록·API 모듈 전략을 담은 구현 계획 문서를 생성한다.
공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "Vue 구현 계획 세워줘", "Vue 화면 구현 순서", "Vue 컴포넌트 설계"
- "Vue 개발 전략 문서 만들어줘", "뷰 개발 계획 수립"
- "Vue 화면 개발 계획", "Vue 구현 전략"

---

## 입력

### 필수
- 화면설계서(SCR-NNN) 목록 — 화면 ID, 화면명, UC 연계 정보
- API 설계서(operationId, 경로, HTTP 메서드) — 연동 API 목록
- 프로젝트 구조 (`ai-dlc-vue-project-setup` 산출물 참조)

### 선택
- 유즈케이스 우선순위 (상위 UC → 구현 우선순위 결정)
- 팀 규모·스프린트 일정

---

## 분석 절차

### 1단계: 화면 목록 정리 및 분류
화면설계서를 읽어 다음 기준으로 분류:
- **화면 유형**: LIST(목록 조회) / FORM(등록·수정) / DETAIL(상세 조회) / DASHBOARD / AUTH(인증)
- **우선순위**: 연계 UC의 Priority(High/Medium/Low) 기준
- **의존 관계**: 공통 컴포넌트를 사용하는 화면 파악

### 2단계: 라우트 구조 설계
- `src/router/index.ts`의 route 배열 설계
- View 컴포넌트 경로 매핑 (`src/views/XxxView.vue`)
- 중첩 라우트 필요 여부 결정 (탭형 화면, 공통 레이아웃)
- 지연 로딩(`() => import(...)`) 적용 대상 결정
- 네비게이션 가드 인증 체크 경로 목록

### 3단계: 컴포넌트 분류
컴포넌트를 3계층으로 분류:

| 계층 | 위치 | 역할 | 예시 |
|:---|:---|:---|:---|
| View | `src/views/` | 라우트 단위 페이지, 데이터 패칭 담당 | `UsersView.vue` |
| Container | `src/components/` | 스토어/Composable 연결, 자식에게 props 전달 | `UserListContainer.vue` |
| Presentational | `src/components/` | 순수 UI, props/emits만 사용 | `UserTable.vue`, `UserForm.vue` |

공통 컴포넌트 추출:
| 유형 | 컴포넌트명 | 사용 화면 |
|:---|:---|:---|
| 레이아웃 | AppLayout, Sidebar, Header | 전체 |
| 데이터 테이블 | DataTable | LIST 화면 |
| 검색 폼 | SearchForm | LIST 화면 |
| 확인 다이얼로그 | ConfirmDialog | FORM 화면 |
| 로딩·에러 | LoadingSpinner, ErrorMessage | 전체 |

### 4단계: Pinia 스토어 목록 결정
- **인증 스토어** (`src/stores/auth.ts`): 토큰, 사용자 정보, login/logout 항상 포함
- **UI 스토어** (`src/stores/ui.ts`): 사이드바 열림·닫힘, 전역 알림 등
- **도메인 스토어**: 서버 데이터 캐싱은 Vue Query 담당 → Pinia는 순수 UI 상태만

스토어 설계 원칙:
- 서버 상태(API 데이터) → Vue Query (`useQuery`, `useMutation`)
- 클라이언트 UI 상태 → Pinia store
- 컴포넌트 로컬 상태 → `ref()`, `reactive()`

### 5단계: Composable 목록 결정
반복 사용되는 로직을 Composable로 추출:
| Composable | 위치 | 역할 |
|:---|:---|:---|
| `useXxxList` | `src/composables/` | 목록 조회 + 필터링 + 페이지네이션 |
| `useXxxForm` | `src/composables/` | 폼 상태 + VeeValidate + 저장 mutation |
| `useXxxDetail` | `src/composables/` | 단건 조회 + 삭제 mutation |
| `useAuth` | `src/composables/` | auth 스토어 + 로그인/로그아웃 래핑 |
| `useToast` | `src/composables/` | 알림 메시지 공통 처리 |

### 6단계: API 모듈 목록 결정
- 도메인별 API 파일 분리: `src/api/user.api.ts`, `src/api/product.api.ts` 등
- 함수명 규칙: `fetchXxxList`, `fetchXxxById`, `createXxx`, `updateXxx`, `deleteXxx`
- Vue Query queryKey 설계: `['users', 'list', filters]` 배열 계층 형식
- 에러 처리: Axios 인터셉터 공통 처리 + 컴포넌트 레벨 에러 표시

### 7단계: 구현 우선순위 결정
1. 공통 레이아웃 + shadcn-vue 기본 컴포넌트
2. 인증 화면 (Login/Logout) + Pinia auth 스토어
3. 우선순위 High 화면 (LIST → DETAIL → FORM 순)
4. 우선순위 Medium/Low 화면
5. e2e 테스트 (`ai-dlc-vue-e2e-test-gen`)

---

## 산출물

- `Vue구현계획_{사업명}_{YYYYMMDD}.md`

### 계획 문서 구성
1. 화면 목록 및 우선순위 표
2. 라우트 구조 표
3. 컴포넌트 분류 표
4. Pinia 스토어 목록 표
5. Composable 목록 표
6. API 모듈 목록 표
7. 구현 순서 (단계별 체크리스트)
8. 기술 스택 및 파일 명명 규칙

template.md에서 계획 문서 마크다운 템플릿을 참조한다.
