---
name: ai-dlc-fe-component-gen
description: AI-DLC 개발단계(프론트엔드-React) 스킬. 화면설계서와 API설계서를 기반으로 React 컴포넌트·Custom Hook·API 클라이언트 코드를 생성한다. "컴포넌트 만들어줘", "화면 구현해줘", "React 컴포넌트 생성", "페이지 코드 만들어줘", "화면 코드 생성", "UI 구현해줘", "리액트 화면 만들어줘", "CRUD 화면 생성" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC React 컴포넌트·Hook·API 코드 생성

화면설계서(SCR-NNN)·API설계서(operationId)·구현 계획을 읽어 페이지 컴포넌트·공통 컴포넌트·Custom Hook·API 클라이언트·Zod 스키마를 생성한다.
공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "컴포넌트 만들어줘", "화면 구현해줘", "React 컴포넌트 생성", "페이지 코드 만들어줘"
- "화면 코드 생성", "UI 구현해줘", "리액트 화면 만들어줘", "CRUD 화면 생성"
- "로그인 화면 만들어줘", "목록 화면 구현해줘", "등록 폼 만들어줘"

---

## 입력

### 필수
- 화면설계서(SCR-NNN): 화면 ID, 화면명, 화면 유형(LIST/FORM/DETAIL), 입력·출력 필드 목록
- API 설계서(operationId): 연동 API 목록, 경로, HTTP 메서드, 요청·응답 스키마

### 선택
- 구현 계획 문서(`ai-dlc-fe-impl-plan` 산출물) — 공통 컴포넌트 목록, 구현 순서 참조
- 기존 프로젝트 구조 (`src/` 디렉터리) — 기존 컴포넌트 재사용 여부 확인

---

## 분석 절차

### 1단계: 화면 유형 파악 및 생성 전략 결정
| 화면 유형 | 생성 파일 | 주요 컴포넌트 |
|:---|:---|:---|
| LIST | XxxListPage.tsx + useXxxList.ts + xxx.api.ts | DataTable, SearchForm, Pagination |
| FORM (등록) | XxxCreatePage.tsx + useCreateXxx.ts | React Hook Form + Zod |
| FORM (수정) | XxxEditPage.tsx + useUpdateXxx.ts | React Hook Form + useQuery 초기값 |
| DETAIL | XxxDetailPage.tsx + useXxxDetail.ts | 읽기 전용 필드 표시 |
| AUTH | LoginPage.tsx + useLoginMutation.ts | 폼 + 토큰 저장 |

### 2단계: TypeScript 타입 정의 (`src/types/`)
화면 I/O 필드 → TypeScript 인터페이스 생성:
```typescript
export interface XxxVO { xxxxxId: number; xxxxxNm: string; ... }
export interface XxxCreateReq { xxxxxNm: string; ... }
export interface XxxFilter { xxxxxNm?: string; page?: number; size?: number }
```

### 3단계: API 클라이언트 함수 (`src/api/xxx.api.ts`)
operationId → Axios 함수 매핑:
- `getXxxList`: GET + 필터 파라미터
- `getXxxById`: GET /:id
- `createXxx`: POST
- `updateXxx`: PUT /:id
- `deleteXxx`: DELETE /:id

### 4단계: queryKey 정의 (`src/api/queryKeys.ts`)
```typescript
export const xxxKeys = {
  all: ['xxxs'] as const,
  list: (filter?: XxxFilter) => [...xxxKeys.all, 'list', filter] as const,
  detail: (id: number) => [...xxxKeys.all, 'detail', id] as const,
}
```

### 5단계: Custom Hook (`src/hooks/`)
- `useXxxList(filter)` — useQuery로 목록 조회
- `useXxxDetail(id)` — useQuery로 단건 조회
- `useCreateXxx()` — useMutation + invalidateQueries
- `useUpdateXxx()` — useMutation + invalidateQueries
- `useDeleteXxx()` — useMutation + invalidateQueries

### 6단계: Zod 스키마 + React Hook Form (`src/types/xxx.schema.ts`)
FORM 화면에 필요한 입력 검증 스키마 생성.

### 7단계: 페이지 컴포넌트 (`src/pages/`)
화면 유형에 맞는 컴포넌트 생성. template.md 골격 참조.

### 8단계: 공통 컴포넌트 (`src/components/`)
여러 화면에서 재사용되는 DataTable, SearchForm 등 공통 컴포넌트.

---

## 코드 생성 원칙

- **TypeScript strict**: 모든 Props에 인터페이스 명시, `any` 금지
- **컴포넌트 규모**: 단일 파일 최대 150줄; 초과 시 하위 컴포넌트 분리
- **Custom Hook 분리**: 상태·비동기 로직은 Hook으로, 컴포넌트는 렌더링에 집중
- **인라인 스타일 금지**: Tailwind 유틸리티 클래스 사용, cn() 함수 조합
- **에러 처리**: isError 상태 처리 + Toast 알림 (shadcn/ui useToast)
- **로딩 처리**: isLoading 상태 → Spinner 컴포넌트
- **접근성**: 버튼에 aria-label, 폼 필드에 htmlFor·id 연결

---

## 산출물

| 파일 경로 | 설명 |
|:---|:---|
| `src/types/{도메인}.types.ts` | TypeScript 인터페이스 |
| `src/types/{도메인}.schema.ts` | Zod 스키마 (FORM 화면) |
| `src/api/{도메인}.api.ts` | Axios API 함수 |
| `src/api/queryKeys.ts` | queryKey 정의 (갱신) |
| `src/hooks/use{Xxx}List.ts` | 목록 조회 Hook |
| `src/hooks/use{Xxx}Detail.ts` | 단건 조회 Hook |
| `src/hooks/use{Create/Update/Delete}Xxx.ts` | 뮤테이션 Hook |
| `src/pages/{Xxx}{유형}Page.tsx` | 페이지 컴포넌트 |
| `src/components/{컴포넌트명}.tsx` | 공통 컴포넌트 |

template.md에서 각 파일의 기본 코드 골격을 참조한다.
