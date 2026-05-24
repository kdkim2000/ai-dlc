---
name: ai-dlc-nxt-page-gen
description: AI-DLC 개발단계(프론트엔드-Next.js) 스킬. Next.js App Router 페이지·레이아웃·서버/클라이언트 컴포넌트 코드를 생성한다. "Next.js 페이지 만들어줘", "App Router 화면 구현", "서버 컴포넌트 코드 생성", "RSC 페이지 생성", "Next.js 화면 코드 작성" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Next.js App Router 페이지·컴포넌트 코드 생성

화면설계서(SCR-NNN)와 구현 계획서(`Next.js구현계획_*.md`)를 입력받아 App Router 기반 페이지·레이아웃·컴포넌트 코드를 생성한다.

## 트리거

- "Next.js 페이지 만들어줘", "App Router 화면 구현"
- "서버 컴포넌트 코드 생성", "RSC 페이지 생성"
- "Next.js 화면 코드 작성", "페이지 컴포넌트 생성"

---

## 입력

### 필수
- 화면설계서 (SCR-NNN) 또는 화면 요구사항 설명
- 라우트 경로 (예: `app/(dashboard)/users/page.tsx`)

### 선택
- `Next.js구현계획_*.md` (RSC/CC 분류 확인용)
- API 설계서 (데이터 패칭 대상 엔드포인트)

---

## 생성 절차

1. **RSC/CC 경계 결정**: 인터랙션·상태·이벤트 필요 여부 확인
2. **라우트 파일 생성 순서**:
   - `page.tsx` (RSC 기본, 데이터 패칭 포함)
   - `layout.tsx` (공유 UI, 네비게이션)
   - `loading.tsx` (Suspense 폴백 스켈레톤)
   - `error.tsx` (에러 경계, `'use client'` 필수)
3. **컴포넌트 분리**:
   - 목록 표시 → RSC (`[Domain]Table`, `[Domain]Card`)
   - 폼·검색·버튼 → CC (`[Domain]Form`, `SearchInput`, `DeleteButton`)
4. **데이터 패칭 패턴 적용**: RSC는 `async function` + `fetch()`, CC는 TanStack Query

---

## 코딩 원칙

| 원칙 | 규칙 |
|:---|:---|
| RSC 기본 | `'use client'` 없는 파일은 자동으로 Server Component |
| 이미지 | `<img>` 금지 → `next/image` 필수 |
| 링크 | `<a href>` 금지 → `next/link` 필수 |
| Metadata | RSC page.tsx에 `export const metadata` 또는 `generateMetadata()` 추가 |
| 컴포넌트 크기 | 파일당 최대 150줄; 초과 시 하위 컴포넌트로 분리 |
| 에러 경계 | 각 Route Segment에 `error.tsx` 필수 작성 |
| 로딩 | 데이터 패칭이 있는 Segment에 `loading.tsx` 작성 |
| Server Action 연동 | 폼의 `action` prop에 Server Action 함수 직접 전달 |

---

## RSC 데이터 패칭 패턴

```typescript
// RSC 목록 페이지 — fetch + revalidate
async function getUsers(searchParams: { page?: string }) {
  const res = await fetch(
    `${process.env.API_BASE_URL}/users?page=${searchParams.page ?? 1}`,
    { next: { revalidate: 60 } }
  )
  if (!res.ok) throw new Error('Failed to fetch users')
  return res.json() as Promise<ApiResponse<User[]>>
}

// RSC 단건 — force-cache
async function getUser(id: string) {
  const res = await fetch(
    `${process.env.API_BASE_URL}/users/${id}`,
    { cache: 'force-cache' }
  )
  if (!res.ok) throw new Error('Failed to fetch user')
  return res.json() as Promise<ApiResponse<User>>
}
```

---

## CC 데이터 패칭 패턴 (TanStack Query)

```typescript
'use client'

import { useQuery } from '@tanstack/react-query'

export function UserSearch() {
  const [keyword, setKeyword] = useState('')
  const { data, isLoading } = useQuery({
    queryKey: ['users', 'search', keyword],
    queryFn: () => fetchUsers({ keyword }),
    enabled: keyword.length > 1,
  })
  // ...
}
```

---

## Server Action 폼 연동 패턴

```typescript
// CC 폼 컴포넌트
'use client'

import { useFormState, useFormStatus } from 'react-dom'
import { createUser } from '@/actions/user'

export function UserForm() {
  const [state, action] = useFormState(createUser, { error: null })
  return (
    <form action={action}>
      {/* 입력 필드 */}
      <SubmitButton />
      {state.error && <p className="text-destructive">{state.error}</p>}
    </form>
  )
}

function SubmitButton() {
  const { pending } = useFormStatus()
  return <Button type="submit" disabled={pending}>{pending ? '처리중...' : '저장'}</Button>
}
```

---

## 생성 파일 목록

| 파일 | 분류 | 설명 |
|:---|:---:|:---|
| `app/(dashboard)/[domain]/page.tsx` | RSC | 목록 페이지 (데이터 패칭 + Suspense) |
| `app/(dashboard)/[domain]/layout.tsx` | RSC/CC | 레이아웃 (헤더, 탭) |
| `app/(dashboard)/[domain]/loading.tsx` | RSC | 스켈레톤 UI |
| `app/(dashboard)/[domain]/error.tsx` | CC | 에러 경계 (`'use client'`) |
| `app/(dashboard)/[domain]/new/page.tsx` | RSC | 등록 페이지 껍데기 |
| `app/(dashboard)/[domain]/[id]/page.tsx` | RSC | 상세 페이지 |
| `app/(dashboard)/[domain]/[id]/edit/page.tsx` | RSC | 수정 페이지 껍데기 |
| `components/[Domain]Table.tsx` | RSC | 목록 테이블 |
| `components/[Domain]Form.tsx` | CC | 등록·수정 폼 |
| `components/[Domain]DeleteButton.tsx` | CC | 삭제 버튼 + Dialog |

---

## 산출물

- `app/**/*.tsx` — 페이지·레이아웃·로딩·에러 컴포넌트
- `components/**/*.tsx` — 재사용 UI 컴포넌트
