---
name: ai-dlc-fe-impl-plan
description: AI-DLC 개발단계(프론트엔드-React) 스킬. 화면설계서와 API설계서를 기반으로 프론트엔드 구현 전략 계획 문서를 생성한다. "프론트엔드 구현 계획 세워줘", "컴포넌트 구현 전략", "화면 구현 계획", "React 구현 전략", "UI 개발 계획서 만들어줘", "프론트엔드 개발 순서 정해줘", "화면 개발 계획 수립" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC 프론트엔드 구현 전략 계획

화면설계서(SCR-NNN)·API설계서·프로젝트 구조를 분석하여 컴포넌트 구현 순서·공통 컴포넌트 목록·상태 관리 전략·API 연동 전략을 담은 계획 문서를 생성한다.
공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "프론트엔드 구현 계획 세워줘", "컴포넌트 구현 전략", "화면 구현 계획"
- "React 구현 전략", "UI 개발 계획서 만들어줘", "프론트엔드 개발 순서 정해줘"
- "화면 개발 계획 수립", "React 개발 전략 문서 만들어줘"

---

## 입력

### 필수
- 화면설계서(SCR-NNN) 목록 — 화면 ID, 화면명, UC 연계 정보
- API 설계서(operationId, 경로, HTTP 메서드) — 연동 API 목록
- 프로젝트 구조 (`ai-dlc-fe-project-setup` 산출물 참조)

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

### 2단계: 공통 컴포넌트 추출
여러 화면에서 반복 사용되는 패턴을 공통 컴포넌트로 추출:
| 유형 | 컴포넌트명 | 사용 화면 |
|:---|:---|:---|
| 레이아웃 | Layout, Sidebar, Header | 전체 |
| 데이터 테이블 | DataTable | LIST 화면 |
| 검색 폼 | SearchForm | LIST 화면 |
| 모달 | ConfirmDialog | FORM 화면 |
| 폼 필드 | FormField, FormSelect | FORM 화면 |
| 로딩·에러 | Spinner, ErrorBoundary | 전체 |

### 3단계: 화면별 구현 순서 결정
의존성 없는 화면 → 공통 컴포넌트 → 의존성 있는 화면 순서로 정렬:
1. 공통 레이아웃 + shadcn/ui 기본 컴포넌트
2. 인증 화면(Login/Logout)
3. 우선순위 High 화면(LIST → DETAIL → FORM 순)
4. 우선순위 Medium/Low 화면

### 4단계: 상태 관리 전략 결정
- **서버 상태** (API 데이터): TanStack Query (`useQuery`, `useMutation`) 사용
- **클라이언트 상태** (UI 상태, 사용자 정보): Zustand store 사용
- **로컬 상태** (폼·모달·토글): `useState` / `useReducer` 사용
- Zustand store 도메인 분리: `useAuthStore`, `useUserStore` 등

### 5단계: API 연동 전략 정리
- Axios 인스턴스 공유 (`src/lib/axios.ts`)
- API 함수 도메인별 파일 분리 (`src/api/user.api.ts` 등)
- React Query queryKey 설계: `['users', 'list', filters]` 배열 형식
- 에러 처리: 인터셉터 공통 처리 + 컴포넌트 레벨 에러 표시

---

## 산출물

- `프론트엔드구현계획_{사업명}_{YYYYMMDD}.md`

### 계획 문서 구성
1. 화면 목록 및 우선순위 표
2. 공통 컴포넌트 목록
3. 화면별 구현 순서 (단계별)
4. 상태 관리 전략
5. API 연동 전략
6. 기술 스택 및 파일 명명 규칙

template.md에서 계획 문서 마크다운 템플릿을 참조한다.
