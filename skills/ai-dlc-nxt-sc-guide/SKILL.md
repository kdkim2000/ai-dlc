---
name: ai-dlc-nxt-sc-guide
description: AI-DLC 개발단계(프론트엔드-Next.js) 스킬. Server Components 데이터 패칭·캐싱·스트리밍 가이드를 제공한다. "Server Component 사용법", "RSC 데이터 패칭", "Next.js fetch 캐싱", "Suspense 스트리밍", "use cache 사용법", "서버 컴포넌트 패턴" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Next.js Server Components 가이드

## 트리거

- "Server Component 사용법", "RSC 데이터 패칭"
- "Next.js fetch 캐싱", "Suspense 스트리밍"
- "use cache 사용법", "서버 컴포넌트 패턴"

---

## RSC / CC 경계 판단 기준

| 조건 | 권장 |
|:---|:---|
| 데이터베이스 직접 조회 | RSC |
| SEO가 중요한 페이지 | RSC |
| 정적 텍스트·목록 표시 | RSC |
| useState, useReducer 사용 | CC |
| onClick, onChange 등 이벤트 핸들러 | CC |
| useEffect, useRef 사용 | CC |
| localStorage, window 접근 | CC |
| TanStack Query useQuery/useMutation | CC |
| Toast, Dialog 등 UI 상태 | CC |

**원칙**: RSC가 기본값. 위 CC 조건 중 하나라도 해당하면 `'use client'` 추가.

---

## fetch() 캐싱 옵션

```typescript
// 1. 요청당 캐시 (기본값) — 빌드 시점에 캐시, ISR
fetch(url, { cache: 'force-cache' })

// 2. 시간 기반 재검증 — 60초마다 백그라운드 갱신
fetch(url, { next: { revalidate: 60 } })

// 3. 캐시 없음 — 항상 최신 (SSR)
fetch(url, { cache: 'no-store' })

// 4. 태그 기반 무효화
fetch(url, { next: { tags: ['users'] } })
// Server Action에서: revalidateTag('users')
```

---

## Parallel Data Fetching (병렬 패칭)

```typescript
// 순차 X (느림)
const user = await getUser(id)
const posts = await getPosts(id)

// 병렬 O (빠름)
const [user, posts] = await Promise.all([
  getUser(id),
  getPosts(id),
])
```

---

## Suspense 스트리밍

```typescript
// app/users/page.tsx
import { Suspense } from 'react'

export default function UsersPage() {
  return (
    <div>
      <h1>사용자 목록</h1>
      {/* 빠른 UI 먼저 표시, 느린 데이터는 나중에 스트리밍 */}
      <Suspense fallback={<UserTableSkeleton />}>
        <UserTableWrapper />  {/* async RSC */}
      </Suspense>
    </div>
  )
}
```

---

## React cache() — 요청 내 중복 제거

```typescript
import { cache } from 'react'

// 같은 요청에서 같은 id로 두 번 호출해도 fetch 1회만 실행
export const getUser = cache(async (id: string) => {
  const res = await fetch(`${process.env.API_BASE_URL}/users/${id}`)
  return res.json()
})
```

---

## Next.js 15 `use cache` 지시어 (실험적)

```typescript
// 함수 수준 캐싱 (Next.js 15 experimental)
async function getUsers() {
  'use cache'
  const res = await fetch(`${process.env.API_BASE_URL}/users`)
  return res.json()
}
```

---

## 서버에서만 실행되어야 하는 코드 격리

```typescript
import 'server-only'  // CC에서 import 시 빌드 오류 발생

export async function getSecretData() {
  // DB 조회, 비밀 환경 변수 접근 등
}
```

---

## 데이터 패칭 선택 가이드

| 상황 | 전략 |
|:---|:---|
| SEO 필요, 정적 목록 | RSC + `force-cache` |
| 주기적 갱신 목록 | RSC + `revalidate: N` |
| 사용자 요청마다 최신 | RSC + `no-store` |
| 클라이언트 검색·필터 | CC + TanStack Query `useQuery` |
| 폼 제출·수정 | Server Actions |
| 외부 노출 API | Route Handlers |
