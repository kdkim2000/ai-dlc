---
name: ai-dlc-fe-lint-check
description: AI-DLC 개발단계(프론트엔드-React) 스킬. ESLint 코드 스타일 규칙 준수 여부를 정적 분석한다. "ESLint 검사해줘", "코드 스타일 확인", "린트 오류 찾아줘", "코딩 컨벤션 검사", "Prettier 포맷 확인", "린트 규칙 체크", "코드 스타일 검사" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC ESLint 코드 스타일 검사

React/TypeScript 소스코드에서 ESLint·Prettier·Import 순서·파일명 컨벤션 위반 패턴을 정적 분석하고 검사 결과를 보고서로 출력한다. 파일을 수정하지 않는다.

## 트리거

- "ESLint 검사해줘", "코드 스타일 확인", "린트 오류 찾아줘"
- "코딩 컨벤션 검사", "Prettier 포맷 확인", "린트 규칙 체크"
- "코드 스타일 검사", "ESLint 규칙 위반 찾아줘"

---

## 입력

- 검사 대상 파일 경로 또는 디렉터리 (기본: `src/`)

---

## 검사 항목

### LN-001: react-hooks/exhaustive-deps 위반
```typescript
// 위반 — 의존성 배열에 사용된 변수 누락
useEffect(() => {
  fetchUser(userId)  // userId가 의존성 배열에 없음
}, [])  // ← 위반

// 권장
useEffect(() => {
  fetchUser(userId)
}, [userId, fetchUser])
```

### LN-002: 미사용 변수·Import
```typescript
// 위반
import { useState, useEffect, useCallback } from 'react'  // useCallback 미사용
const unusedVar = 'hello'  // 미사용 변수
```

### LN-003: `no-explicit-any` 위반
```typescript
// 위반 — .eslintrc에 error 설정된 경우
const data: any = {}
```
> TC-001과 중복이나 ESLint 관점에서 별도 검사

### LN-004: Import 순서 위반
```typescript
// 권장 순서: 외부 라이브러리 → 내부 절대경로(@/) → 상대경로
import { useState } from 'react'         // 1. 외부
import { useUserList } from '@/hooks'    // 2. 내부 절대경로
import UserCard from './UserCard'         // 3. 상대경로
```

### LN-005: 파일명 컨벤션 위반
| 파일 유형 | 컨벤션 | 위반 예 |
|:---|:---|:---|
| 컴포넌트 (.tsx) | PascalCase | `userList.tsx` → `UserListPage.tsx` |
| Hook (.ts) | camelCase, use 접두사 | `UserList.ts` → `useUserList.ts` |
| API 함수 (.ts) | camelCase + .api 접미사 | `User.ts` → `user.api.ts` |
| 타입 파일 (.ts) | camelCase + .types 접미사 | `User.ts` → `user.types.ts` |

### LN-006: console.log 잔존
```typescript
// 위반 — 프로덕션 코드에 console.log 잔존
console.log('디버깅용 로그')
console.log(data)
```

### LN-007: Prettier 포맷팅 불일치
```typescript
// 위반 — singleQuote, semi 규칙 위반
const name = "홍길동";  // 큰따옴표 + 세미콜론

// 권장
const name = '홍길동'  // 작은따옴표 + 세미콜론 없음
```

### LN-008: react-refresh/only-export-components 위반
```typescript
// 위반 — 컴포넌트 파일에서 비컴포넌트를 named export
export const CONSTANT = 'value'  // 컴포넌트 파일에서 상수 export
export default function MyComponent() { ... }

// 권장 — 상수는 별도 파일(constants.ts)로 분리
```

---

## 검사 방법

1. `Glob`으로 `src/**/*.{ts,tsx}` 파일 목록 수집
2. `Grep`으로 패턴 검색:
   - `console\.(log|warn|debug)` — LN-006
   - `"[^"]*"` (큰따옴표 문자열) — LN-007 (Prettier)
   - `useEffect\(.*\[\]` — LN-001 후보 (빈 의존성)
   - `import.*'` 순서 — LN-004
3. `Read`로 파일 내용 읽어 맥락 최종 판정

---

## 보고서 형식

```markdown
# ESLint 코드 스타일 검사 결과

| 항목 | 내용 |
|:---|:---|
| 검사일 | YYYY-MM-DD |
| 검사 범위 | src/ |

## 검사 결과 요약

| 코드 | 항목 | 건수 |
|:---|:---|:---:|
| LN-001 | exhaustive-deps 위반 | N |
| LN-006 | console.log 잔존 | N |

**종합 판정**: 통과 / 조건부통과 / 재검토필요

## 이슈 목록

| VI-ID | 코드 | 파일 | 라인 | 설명 |
|:---|:---|:---|:---:|:---|
| VI-001 | LN-006 | src/hooks/useUserList.ts | 12 | console.log 잔존 — 제거 필요 |
```

---

## 산출물

- `ESLint검사결과_{YYYYMMDD}.md`
