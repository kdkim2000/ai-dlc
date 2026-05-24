---
name: ai-dlc-fe-code-revise
description: AI-DLC 개발단계(프론트엔드-React) 스킬. 코드 리뷰 결과를 소스코드에 반영한다. "코드 수정해줘", "리뷰 결과 반영", "프론트엔드 코드 개선", "지적 사항 반영", "코드 품질 개선", "리뷰 피드백 적용", "코드 리뷰 결과 반영해줘" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC 프론트엔드 코드 수정

코드 리뷰 보고서(`코드품질검토_{YYYYMMDD}.md`) 또는 TypeScript/ESLint 검사 결과의 이슈 목록을 읽어 소스코드에 수정을 반영한다. 최소 변경 원칙을 준수하며, 기능 동작에 영향 없는 리팩터링은 수행하지 않는다.

## 트리거

- "코드 수정해줘", "리뷰 결과 반영", "프론트엔드 코드 개선"
- "지적 사항 반영", "코드 품질 개선", "리뷰 피드백 적용"
- "코드 리뷰 결과 반영해줘", "TS 오류 수정해줘", "ESLint 오류 수정"

---

## 입력

### 필수
- 검토 보고서 파일 (`코드품질검토_*.md` 또는 `TypeScript검사결과_*.md` 또는 `ESLint검사결과_*.md`)

### 선택
- 특정 VI-ID 또는 이슈 코드만 적용 (미지정 시 전체 적용)

---

## 수정 절차

1. `Read`로 보고서 파일 읽어 이슈 목록(VI-NNN) 파악
2. 수정 우선순위 결정:
   - **1순위 SC** — 보안 이슈 (XSS, 인증 누락)
   - **2순위 TC** — TypeScript 타입 오류 (any 사용, null 처리 누락)
   - **3순위 PF** — 성능 안티패턴 (exhaustive-deps, 불필요한 재렌더링)
   - **4순위 LC** — 레이어 설계 위반 (API 직접 호출, 150줄 초과)
   - **5순위 A11Y** — 접근성
   - **6순위 LN** — ESLint 스타일 (console.log, import 순서)
3. 각 이슈별로 `Read`로 대상 파일 읽어 위치 확인
4. `Edit`으로 최소 범위 수정 적용
5. 수정 완료 후 적용 결과 표 출력

---

## 수정 원칙

- **최소 변경**: 이슈 범위 밖 코드는 수정하지 않음
- **기능 보존**: 로직 변경 없이 코드 품질만 개선
- **타입 우선**: `any` 제거 시 실제 타입 확인 후 적용 (추측 금지)
- **console.log**: 즉시 삭제 (디버깅 코드는 보존 필요 시 주석 처리하지 말고 삭제)

---

## 주요 수정 패턴

### SC-001: dangerouslySetInnerHTML 제거
```typescript
// 위반
<div dangerouslySetInnerHTML={{ __html: content }} />

// 수정 — DOMPurify 사용 또는 구조 변경
import DOMPurify from 'dompurify'
<div dangerouslySetInnerHTML={{ __html: DOMPurify.sanitize(content) }} />
```

### TC-001: any 타입 제거
```typescript
// 위반
const data: any = response.data

// 수정 — 실제 타입 적용
const data: UserVO = response.data
```

### TC-003: null 처리 추가
```typescript
// 위반
const name = user.profile.name

// 수정
const name = user.profile?.name ?? ''
```

### PF-001: useEffect 의존성 배열 수정
```typescript
// 위반
useEffect(() => {
  fetchUser(userId)
}, [])

// 수정
useEffect(() => {
  fetchUser(userId)
}, [userId, fetchUser])
```

### LC-002: API 호출을 Custom Hook으로 분리
```typescript
// 위반 — 페이지 컴포넌트 내 직접 호출
useEffect(() => {
  axios.get('/api/users').then(setUsers)
}, [])

// 수정 — Custom Hook 사용
const { data: users } = useUserList()
```

### LN-006: console.log 제거
```typescript
// 위반
console.log('디버그:', data)

// 수정 — 해당 줄 삭제
```

---

## 수정 결과 표

```markdown
## 수정 적용 결과

| VI-ID | 코드 | 파일 | 수정 내용 | 상태 |
|:---|:---|:---|:---|:---:|
| VI-001 | TC-001 | src/api/user.api.ts | `any` → `UserVO` 타입 변경 | ✅ 완료 |
| VI-002 | LN-006 | src/hooks/useUserList.ts | console.log 제거 | ✅ 완료 |
| VI-003 | LC-002 | src/pages/UserListPage.tsx | API 호출 useUserList Hook으로 이동 | ✅ 완료 |

**적용 완료**: N건 / **건너뜀**: N건 (사유 기술)
```

---

## 산출물

- 수정된 소스 파일들 (원본 대체)
- 수정 적용 결과 표 (대화창 출력)
