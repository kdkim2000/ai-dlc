---
name: ai-dlc-nxt-code-revise
description: AI-DLC 개발단계(프론트엔드-Next.js) 스킬. Next.js 코드 리뷰 결과를 반영하여 코드를 수정한다. "Next.js 코드 수정해줘", "RSC 리뷰 결과 반영", "NX 이슈 코드 수정", "Next.js 코드 개선", "코드 품질 개선" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Next.js App Router 코드 수정

`ai-dlc-nxt-code-review` 결과물(`코드품질검토_*.md`)을 입력받아 NX 이슈 코드를 우선순위에 따라 수정한다.

## 트리거

- "Next.js 코드 수정해줘", "RSC 리뷰 결과 반영"
- "NX 이슈 코드 수정", "Next.js 코드 개선"
- "코드 품질 개선", "코드 리뷰 반영해줘"

---

## 입력

### 필수
- `코드품질검토_*.md` (NX 이슈 목록)

### 선택
- 수정 범위 (특정 이슈 코드만 처리 등)

---

## 수정 우선순위

| 우선순위 | 이슈 코드 | 이유 |
|:---:|:---|:---|
| 1 | SC-001 | 보안 취약점 — 즉시 수정 |
| 2 | NX-002, NX-005, NX-006, NX-008 | 아키텍처·보안 오류 |
| 3 | TC-001, TC-003 | 타입 안전성 |
| 4 | NX-001 | RSC 전환 (성능·번들 크기) |
| 5 | NX-003, NX-004 | next/image, next/link 전환 |
| 6 | NX-007, NX-009 | 캐싱·revalidate 누락 |
| 7 | A11Y-001, A11Y-002 | 접근성 |
| 8 | NX-010, TC-002 | 완성도 |

---

## 이슈별 수정 패턴

### NX-001: `'use client'` 과다 → RSC 전환
```typescript
// Before (불필요한 CC)
'use client'
export function UserTable({ users }: { users: User[] }) {
  return <table>...</table>  // 인터랙션 없음
}

// After (RSC)
export function UserTable({ users }: { users: User[] }) {
  return <table>...</table>
}
```

### NX-002: CC에서 서버 로직 분리
```typescript
// Before (CC에서 prisma 직접 호출)
'use client'
import { prisma } from '@/lib/prisma'
export function UserList() {
  const users = await prisma.user.findMany()  // 불가능
}

// After (RSC 또는 Route Handler로 분리)
// RSC page.tsx에서 fetch() 또는 Server Action 호출
```

### NX-003: `<img>` → `next/image`
```typescript
// Before
<img src={user.avatar} alt={user.name} className="w-10 h-10" />

// After
import Image from 'next/image'
<Image src={user.avatar} alt={user.name} width={40} height={40} className="rounded-full" />
```

### NX-004: `<a>` → `next/link`
```typescript
// Before
<a href="/users">사용자 목록</a>

// After
import Link from 'next/link'
<Link href="/users">사용자 목록</Link>
```

### NX-005: Route Handler 인증 추가
```typescript
// After
const session = await auth()
if (!session) {
  return NextResponse.json(errorResponse('인증이 필요합니다', 'UNAUTHORIZED'), { status: 401 })
}
```

### NX-006: Server Action Zod 검사 추가
```typescript
// After
const parsed = Schema.safeParse(Object.fromEntries(formData))
if (!parsed.success) {
  return { error: '입력값을 확인해 주세요', fieldErrors: parsed.error.flatten().fieldErrors }
}
```

### NX-007: fetch 캐시 전략 추가
```typescript
// After (목록 — 60초 캐시)
fetch(url, { next: { revalidate: 60 } })
// After (단건 — force-cache)
fetch(url, { cache: 'force-cache' })
// After (실시간 — no-store)
fetch(url, { cache: 'no-store' })
```

### NX-009: revalidatePath 추가
```typescript
// After (Server Action 성공 후)
revalidatePath('/users')
revalidatePath(`/users/${id}`)
```

---

## 수정 절차

1. `코드품질검토_*.md` 읽어 이슈 목록 파악
2. 우선순위 순서로 파일별 수정 (`Edit` 도구 사용)
3. 한 파일 수정 완료 후 다음 파일 진행
4. 수정 완료 후 수정 내역 요약 출력

---

## 산출물

- 수정된 소스 파일 일체
- 수정 내역 요약 (이슈코드 → 파일:줄 → 수정 내용)
