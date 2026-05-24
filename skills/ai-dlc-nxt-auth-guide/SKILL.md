---
name: ai-dlc-nxt-auth-guide
description: AI-DLC 개발단계(프론트엔드-Next.js) 스킬. Auth.js v5(NextAuth.js) 인증 가이드를 제공한다. "Next.js 인증 방법", "Auth.js 설정", "NextAuth.js v5", "로그인 구현", "세션 관리", "인증 미들웨어" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Next.js Auth.js v5 인증 가이드

## 트리거

- "Next.js 인증 방법", "Auth.js 설정"
- "NextAuth.js v5", "로그인 구현"
- "세션 관리", "인증 미들웨어"

---

## Auth.js v5 기본 설정

```typescript
// src/lib/auth.ts
import NextAuth from 'next-auth'
import CredentialsProvider from 'next-auth/providers/credentials'
import { z } from 'zod'

const loginSchema = z.object({
  username: z.string().min(1),
  password: z.string().min(1),
})

export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [
    CredentialsProvider({
      name: 'credentials',
      credentials: {
        username: { label: '아이디', type: 'text' },
        password: { label: '비밀번호', type: 'password' },
      },
      async authorize(credentials) {
        const parsed = loginSchema.safeParse(credentials)
        if (!parsed.success) return null

        // TODO: 실제 사용자 조회로 교체
        const user = await validateUser(parsed.data.username, parsed.data.password)
        return user ?? null
      },
    }),
  ],
  pages: { signIn: '/login' },
  callbacks: {
    authorized({ auth, request: { nextUrl } }) {
      const isLoggedIn = !!auth?.user
      const isPublicPath = ['/login', '/register'].includes(nextUrl.pathname)
      if (!isLoggedIn && !isPublicPath) return false
      return true
    },
    jwt({ token, user }) {
      if (user) {
        token.id = user.id
        token.role = (user as { role?: string }).role
      }
      return token
    },
    session({ session, token }) {
      if (token.id) session.user.id = token.id as string
      if (token.role) (session.user as { role?: string }).role = token.role as string
      return session
    },
  },
})
```

---

## Route Handler 등록

```typescript
// app/api/auth/[...nextauth]/route.ts
import { handlers } from '@/lib/auth'
export const { GET, POST } = handlers
```

---

## Middleware 설정

```typescript
// src/middleware.ts
export { auth as middleware } from '@/lib/auth'

export const config = {
  matcher: [
    '/((?!api|_next/static|_next/image|favicon.ico|login|register).*)',
  ],
}
```

---

## Session 접근 방법

### Server Component (RSC) — 서버에서 직접

```typescript
import { auth } from '@/lib/auth'

export default async function ProfilePage() {
  const session = await auth()
  if (!session) redirect('/login')

  return <div>안녕하세요, {session.user.name}님</div>
}
```

### Client Component (CC) — `useSession` 훅

```typescript
'use client'

import { useSession } from 'next-auth/react'

export function UserMenu() {
  const { data: session, status } = useSession()

  if (status === 'loading') return <Skeleton />
  if (!session) return <Link href="/login">로그인</Link>

  return <span>{session.user.name}</span>
}
```

---

## 로그인 페이지 패턴

```typescript
// app/(auth)/login/page.tsx
'use client'

import { signIn } from 'next-auth/react'
import { useRouter } from 'next/navigation'
import { useState } from 'react'

export default function LoginPage() {
  const [error, setError] = useState<string | null>(null)
  const router = useRouter()

  async function handleSubmit(e: React.FormEvent<HTMLFormElement>) {
    e.preventDefault()
    const formData = new FormData(e.currentTarget)

    const result = await signIn('credentials', {
      username: formData.get('username'),
      password: formData.get('password'),
      redirect: false,
    })

    if (result?.error) {
      setError('아이디 또는 비밀번호가 올바르지 않습니다')
    } else {
      router.push('/')
      router.refresh()
    }
  }

  return (
    <form onSubmit={handleSubmit}>
      {error && <p className="text-destructive">{error}</p>}
      <input name="username" type="text" placeholder="아이디" />
      <input name="password" type="password" placeholder="비밀번호" />
      <button type="submit">로그인</button>
    </form>
  )
}
```

---

## 로그아웃

```typescript
// Server Action
'use server'
import { signOut } from '@/lib/auth'

export async function logout() {
  await signOut({ redirectTo: '/login' })
}

// CC 버튼
import { logout } from '@/actions/auth'
<form action={logout}>
  <button type="submit">로그아웃</button>
</form>
```

---

## TypeScript 세션 타입 확장

```typescript
// src/types/next-auth.d.ts
import { DefaultSession } from 'next-auth'

declare module 'next-auth' {
  interface Session {
    user: DefaultSession['user'] & {
      id: string
      role: 'ADMIN' | 'USER'
    }
  }
}
```

---

## 역할 기반 접근 제어

```typescript
// Route Handler에서 역할 확인
const session = await auth()
if (!session) return NextResponse.json(errorResponse('UNAUTHORIZED'), { status: 401 })
if (session.user.role !== 'ADMIN') return NextResponse.json(errorResponse('FORBIDDEN'), { status: 403 })
```
