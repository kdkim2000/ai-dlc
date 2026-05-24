---
name: ai-dlc-fe-ts-check
description: AI-DLC 개발단계(프론트엔드-React) 스킬. TypeScript 타입 안전성을 정적 분석한다. "TypeScript 검사해줘", "타입 오류 찾아줘", "TS 타입 검토", "타입 안전성 확인", "any 타입 찾아줘", "TypeScript 타입 점검", "타입 체크해줘" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC TypeScript 타입 안전성 검사

React/TypeScript 소스코드에서 타입 안전성 위반 패턴을 정적 분석하고 검사 결과를 보고서로 출력한다. 파일을 수정하지 않는다.

## 트리거

- "TypeScript 검사해줘", "타입 오류 찾아줘", "TS 타입 검토"
- "타입 안전성 확인", "any 타입 찾아줘", "TypeScript 타입 점검"
- "타입 체크해줘", "타입스크립트 오류 확인"

---

## 입력

- 검사 대상 파일 경로 또는 디렉터리 (기본: `src/`)

---

## 검사 항목

### TC-001: 암시적·명시적 `any` 사용
```typescript
// 위반 — 검색 패턴: `: any`, `as any`, `<any>`
const data: any = response.data
function process(input: any) { ... }

// 권장
const data: UserVO = response.data
function process(input: UserVO) { ... }
```

### TC-002: 타입 단언(`as`) 과다 사용
```typescript
// 주의 — 타입 좁히기(type guard)로 대체 가능한 경우
const user = data as UserVO  // API 응답이 이미 타입화 된 경우 불필요

// Non-null assertion 남용
const el = document.getElementById('root')!  // 없을 경우 런타임 오류
```

### TC-003: null/undefined 처리 누락
```typescript
// 위반 — optional chaining 미사용
const name = user.profile.name  // profile이 undefined면 런타임 오류

// 권장
const name = user.profile?.name ?? '이름 없음'
```

### TC-004: 함수 반환 타입 미명시 (추론 불가능한 경우)
```typescript
// 위반 — 반환 타입이 불명확한 함수
function getStatus(code: string) {
  if (code === 'A') return { active: true }
  return null
}

// 권장
function getStatus(code: string): { active: boolean } | null { ... }
```

### TC-005: Props 인터페이스 미정의
```typescript
// 위반 — 인라인 또는 미정의 Props
function UserCard({ name, email }: { name: string; email: string }) { ... }

// 권장 — 별도 인터페이스 정의
interface UserCardProps { name: string; email: string }
function UserCard({ name, email }: UserCardProps) { ... }
```

### TC-006: Enum 대신 as const 사용 권장
```typescript
// 주의 — TypeScript enum은 번들 크기·트리쉐이킹 문제
enum Status { Active = 'ACTIVE', Inactive = 'INACTIVE' }

// 권장 — as const 객체
const Status = { Active: 'ACTIVE', Inactive: 'INACTIVE' } as const
type Status = typeof Status[keyof typeof Status]
```

### TC-007: React 관련 타입
```typescript
// 위반
const handler = (e) => { ... }  // 이벤트 타입 미명시

// 권장
const handler = (e: React.ChangeEvent<HTMLInputElement>) => { ... }
const ref = useRef<HTMLDivElement>(null)
const [state, setState] = useState<UserVO | null>(null)
```

---

## 검사 방법

1. `Glob`으로 `src/**/*.{ts,tsx}` 파일 목록 수집
2. `Grep`으로 위반 패턴 검색:
   - `any` 패턴: `: any`, `as any`, `<any>`
   - Non-null assertion: 과다 사용 여부
   - 이벤트 핸들러 타입: `(e) =>` (타입 미명시)
3. 각 파일 Read로 맥락 확인 후 위반 여부 최종 판정

---

## 보고서 형식

```markdown
# TypeScript 타입 검사 결과

| 항목 | 내용 |
|:---|:---|
| 검사일 | YYYY-MM-DD |
| 검사 범위 | src/ |

## 검사 결과 요약

| 코드 | 항목 | 건수 |
|:---|:---|:---:|
| TC-001 | any 타입 사용 | N |
| TC-002 | 타입 단언 과다 | N |
| TC-003 | null 처리 누락 | N |

**종합 판정**: 통과 / 조건부통과 / 재검토필요

## 이슈 목록

| VI-ID | 코드 | 파일 | 라인 | 설명 |
|:---|:---|:---|:---:|:---|
| VI-001 | TC-001 | src/api/user.api.ts | 8 | 응답 타입 `any` 사용 — `UserVO` 타입으로 변경 필요 |
```

---

## 산출물

- `TypeScript검사결과_{YYYYMMDD}.md`
