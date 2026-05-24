---
name: ai-dlc-nxt-middleware-guide
description: AI-DLC 개발단계(프론트엔드-Next.js) 스킬. Next.js 미들웨어 가이드를 제공한다. "Next.js 미들웨어", "middleware.ts 설정", "라우트 보호", "인증 미들웨어", "리다이렉트 미들웨어", "i18n 미들웨어" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Next.js Middleware 가이드

## 트리거

- "Next.js 미들웨어", "middleware.ts 설정"
- "라우트 보호", "인증 미들웨어"
- "리다이렉트 미들웨어", "i18n 미들웨어"

---

## middleware.ts 위치 및 기본 구조

`src/middleware.ts` (또는 프로젝트 루트 `middleware.ts`)에 위치한다. 프로젝트에 하나만 존재 가능.

```typescript
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  // 요청 처리 로직
  return NextResponse.next()
}

export const config = {
  // 미들웨어가 실행될 경로 패턴
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
}
```

---

## Auth.js 인증 미들웨어 (권장 패턴)

```typescript
// src/middleware.ts — Auth.js 위임 방식
export { auth as middleware } from '@/lib/auth'

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico|login|register).*)'],
}
```

---

## 커스텀 인증 미들웨어

```typescript
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'
import { auth } from '@/lib/auth'

export async function middleware(request: NextRequest) {
  const session = await auth()
  const { pathname } = request.nextUrl

  const publicPaths = ['/login', '/register', '/api/auth']
  const isPublic = publicPaths.some((p) => pathname.startsWith(p))

  if (!session && !isPublic) {
    const loginUrl = new URL('/login', request.url)
    loginUrl.searchParams.set('callbackUrl', pathname)
    return NextResponse.redirect(loginUrl)
  }

  // 역할 기반 접근 제어
  if (pathname.startsWith('/admin') && session?.user?.role !== 'ADMIN') {
    return NextResponse.redirect(new URL('/unauthorized', request.url))
  }

  return NextResponse.next()
}
```

---

## 리다이렉트 패턴

```typescript
// 도메인별 리다이렉트
export function middleware(request: NextRequest) {
  const hostname = request.headers.get('host')

  if (hostname?.startsWith('old.')) {
    return NextResponse.redirect(new URL(request.url.replace('old.', 'new.')))
  }

  return NextResponse.next()
}
```

---

## 요청 헤더 수정 (RSC에 데이터 전달)

```typescript
export function middleware(request: NextRequest) {
  const requestHeaders = new Headers(request.headers)
  requestHeaders.set('x-pathname', request.nextUrl.pathname)

  return NextResponse.next({
    request: { headers: requestHeaders },
  })
}

// RSC page.tsx에서 읽기
import { headers } from 'next/headers'
const pathname = headers().get('x-pathname')
```

---

## matcher 패턴 작성 규칙

```typescript
export const config = {
  matcher: [
    // 특정 경로만
    '/dashboard/:path*',
    '/admin/:path*',

    // 정적 파일·API 제외
    '/((?!api|_next/static|_next/image|favicon.ico).*)',

    // 모든 경로 (정적 파일 포함 시 성능 저하 주의)
    '/:path*',
  ],
}
```

---

## Edge Runtime 제약사항

미들웨어는 Edge Runtime에서 실행된다:
- `fs`, `path`, `crypto` 모듈 사용 불가
- `prisma` 직접 호출 불가 (DB 연결 불가)
- Prisma 대신 Auth.js 세션 쿠키 확인으로 인증 처리
- `fetch()` 사용 가능 (외부 API 호출 가능)
- 응답 시간은 최대한 짧게 유지 (50ms 미만 권장)
