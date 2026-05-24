# Next.js 15 App Router 프로젝트 설정 템플릿

> **⚠️ 중요 변경 이력 (2026-05-23)**
> - Tailwind CSS v4로 업그레이드 (`@tailwindcss/postcss` 사용)
> - shadcn/ui nova 프리셋 → `toast` 대신 `sonner` 사용
> - `lucide-react` v1.x 이상 (React 19 호환)
> - `Inter` 폰트 → `Geist` 폰트 (nova 프리셋 기본)
> - `dotenv/config` → `config({ path: '.env.local' })` 명시 필요

---

## package.json

```json
{
  "name": "my-nextjs-app",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "next build",
    "start": "next start",
    "lint": "next lint"
  },
  "dependencies": {
    "next": "^15.0.0",
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "next-auth": "^5.0.0-beta",
    "next-themes": "^0.4.0",
    "zod": "^3.22.0",
    "react-hook-form": "^7.51.0",
    "@hookform/resolvers": "^3.3.0",
    "clsx": "^2.1.0",
    "tailwind-merge": "^3.0.0",
    "class-variance-authority": "^0.7.0",
    "lucide-react": "^1.0.0",
    "sonner": "^2.0.0",
    "tw-animate-css": "^1.4.0",
    "radix-ui": "^1.4.0"
  },
  "devDependencies": {
    "typescript": "^5.4.0",
    "@types/node": "^20.0.0",
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "tailwindcss": "latest",
    "@tailwindcss/postcss": "latest",
    "postcss": "^8.4.0",
    "dotenv": "^16.0.0",
    "eslint": "^8.57.0",
    "eslint-config-next": "^15.0.0",
    "@typescript-eslint/eslint-plugin": "^7.0.0",
    "@typescript-eslint/parser": "^7.0.0",
    "prettier": "^3.2.0",
    "prettier-plugin-tailwindcss": "^0.5.0",
    "shadcn": "^4.8.0"
  }
}
```

> **ORM 선택**:
> - Drizzle ORM: `drizzle-orm`, `@neondatabase/serverless` (Neon Postgres) 또는 `drizzle-orm`, `better-sqlite3`
> - Prisma: `@prisma/client` (dep) + `prisma` (devDep)
> DB 선택에 따라 package.json에 추가한다.

---

## .npmrc

```
legacy-peer-deps=true
```

> React 19 + 일부 패키지의 peer dependency 충돌 방지용.

---

## next.config.ts

```typescript
import type { NextConfig } from 'next'

const nextConfig: NextConfig = {
  experimental: {
    ppr: false,
  },
  images: {
    remotePatterns: [],
  },
}

export default nextConfig
```

---

## tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [{ "name": "next" }],
    "paths": {
      "@/*": ["./src/*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

---

## postcss.config.js

```javascript
module.exports = {
  plugins: {
    '@tailwindcss/postcss': {},
  },
}
```

> **⚠️ Tailwind v4** 에서는 `tailwindcss: {}` + `autoprefixer: {}` 방식이 아닌
> `@tailwindcss/postcss` 단일 플러그인을 사용한다. autoprefixer는 내장됨.

---

## tailwind.config.ts

```typescript
import type { Config } from 'tailwindcss'

const config: Config = {
  darkMode: ['class'],
  content: [
    './src/pages/**/*.{ts,tsx}',
    './src/components/**/*.{ts,tsx}',
    './src/app/**/*.{ts,tsx}',
  ],
}

export default config
```

> **⚠️ Tailwind v4** 에서는 `theme.extend.colors`, `borderRadius`, `keyframes`,
> `plugins` 등을 config 파일에 쓰지 않는다. 색상·애니메이션은 `globals.css`의
> `@theme inline`과 `@import "tw-animate-css"`로 관리한다.

---

## src/app/globals.css

```css
@import "tailwindcss";
@import "tw-animate-css";

@custom-variant dark (&.dark);

@theme inline {
  --color-background: var(--background);
  --color-foreground: var(--foreground);
  --color-card: var(--card);
  --color-card-foreground: var(--card-foreground);
  --color-popover: var(--popover);
  --color-popover-foreground: var(--popover-foreground);
  --color-primary: var(--primary);
  --color-primary-foreground: var(--primary-foreground);
  --color-secondary: var(--secondary);
  --color-secondary-foreground: var(--secondary-foreground);
  --color-muted: var(--muted);
  --color-muted-foreground: var(--muted-foreground);
  --color-accent: var(--accent);
  --color-accent-foreground: var(--accent-foreground);
  --color-destructive: var(--destructive);
  --color-border: var(--border);
  --color-input: var(--input);
  --color-ring: var(--ring);
  --color-sidebar: var(--sidebar);
  --color-sidebar-foreground: var(--sidebar-foreground);
  --color-sidebar-primary: var(--sidebar-primary);
  --color-sidebar-primary-foreground: var(--sidebar-primary-foreground);
  --color-sidebar-accent: var(--sidebar-accent);
  --color-sidebar-accent-foreground: var(--sidebar-accent-foreground);
  --color-sidebar-border: var(--sidebar-border);
  --color-sidebar-ring: var(--sidebar-ring);
  --radius-sm: calc(var(--radius) - 4px);
  --radius-md: calc(var(--radius) - 2px);
  --radius-lg: var(--radius);
  --radius-xl: calc(var(--radius) + 4px);
  --font-sans: var(--font-geist-sans, ui-sans-serif, system-ui, sans-serif);
}

:root {
  --background: oklch(1 0 0);
  --foreground: oklch(0.145 0 0);
  --card: oklch(1 0 0);
  --card-foreground: oklch(0.145 0 0);
  --popover: oklch(1 0 0);
  --popover-foreground: oklch(0.145 0 0);
  --primary: oklch(0.205 0 0);
  --primary-foreground: oklch(0.985 0 0);
  --secondary: oklch(0.97 0 0);
  --secondary-foreground: oklch(0.205 0 0);
  --muted: oklch(0.97 0 0);
  --muted-foreground: oklch(0.556 0 0);
  --accent: oklch(0.97 0 0);
  --accent-foreground: oklch(0.205 0 0);
  --destructive: oklch(0.577 0.245 27.325);
  --border: oklch(0.922 0 0);
  --input: oklch(0.922 0 0);
  --ring: oklch(0.708 0 0);
  --radius: 0.625rem;
  --sidebar: oklch(0.985 0 0);
  --sidebar-foreground: oklch(0.145 0 0);
  --sidebar-primary: oklch(0.205 0 0);
  --sidebar-primary-foreground: oklch(0.985 0 0);
  --sidebar-accent: oklch(0.97 0 0);
  --sidebar-accent-foreground: oklch(0.205 0 0);
  --sidebar-border: oklch(0.922 0 0);
  --sidebar-ring: oklch(0.708 0 0);
}

.dark {
  --background: oklch(0.145 0 0);
  --foreground: oklch(0.985 0 0);
  --card: oklch(0.205 0 0);
  --card-foreground: oklch(0.985 0 0);
  --popover: oklch(0.205 0 0);
  --popover-foreground: oklch(0.985 0 0);
  --primary: oklch(0.922 0 0);
  --primary-foreground: oklch(0.205 0 0);
  --secondary: oklch(0.269 0 0);
  --secondary-foreground: oklch(0.985 0 0);
  --muted: oklch(0.269 0 0);
  --muted-foreground: oklch(0.708 0 0);
  --accent: oklch(0.269 0 0);
  --accent-foreground: oklch(0.985 0 0);
  --destructive: oklch(0.704 0.191 22.216);
  --border: oklch(1 0 0 / 10%);
  --input: oklch(1 0 0 / 15%);
  --ring: oklch(0.556 0 0);
  --sidebar: oklch(0.205 0 0);
  --sidebar-foreground: oklch(0.985 0 0);
  --sidebar-primary: oklch(0.488 0.243 264.376);
  --sidebar-primary-foreground: oklch(0.985 0 0);
  --sidebar-accent: oklch(0.269 0 0);
  --sidebar-accent-foreground: oklch(0.985 0 0);
  --sidebar-border: oklch(1 0 0 / 10%);
  --sidebar-ring: oklch(0.556 0 0);
}

@layer base {
  * {
    @apply border-border;
  }

  body {
    @apply bg-background text-foreground;
  }

  html {
    @apply font-sans;
  }
}
```

> **⚠️ 핵심 구조**:
> 1. `@import "tailwindcss"` — v4 핵심 (구 `@tailwind base/components/utilities` 대체)
> 2. `@import "tw-animate-css"` — 애니메이션 유틸리티
> 3. `@theme inline { --color-*: var(--*) }` — CSS 변수를 Tailwind 유틸리티로 등록
>    이 블록 없이는 `@apply border-border` 같은 구문이 "unknown utility" 오류 발생
> 4. `:root` / `.dark` — 런타임 CSS 변수값 (oklch 색상 체계)
> 5. `@layer base` — Tailwind 유틸리티 적용 (위의 `@theme inline` 이후에 처리됨)

---

## src/lib/utils.ts

```typescript
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}

export function formatDate(date: Date | string): string {
  return new Date(date).toLocaleDateString('ko-KR', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
  })
}

export function formatDateTime(date: Date | string): string {
  return new Date(date).toLocaleString('ko-KR', {
    year: 'numeric',
    month: '2-digit',
    day: '2-digit',
    hour: '2-digit',
    minute: '2-digit',
  })
}
```

> **⚠️ 주의**: `npx shadcn@latest init` 실행 시 `utils.ts`를 `cn()` 함수만 남기고
> 덮어쓴다. init 후 `formatDate`, `formatDateTime` 등 프로젝트 헬퍼를 재추가해야 한다.

---

## src/lib/auth.ts (Auth.js v5)

```typescript
import NextAuth from 'next-auth'
import CredentialsProvider from 'next-auth/providers/credentials'
import { z } from 'zod'

const loginSchema = z.object({
  email: z.string().email(),
  password: z.string().min(1),
})

export const { handlers, signIn, signOut, auth } = NextAuth({
  providers: [
    CredentialsProvider({
      name: 'credentials',
      credentials: {
        email: { label: '이메일', type: 'email' },
        password: { label: '비밀번호', type: 'password' },
      },
      async authorize(credentials) {
        const parsed = loginSchema.safeParse(credentials)
        if (!parsed.success) return null

        // TODO: DB 조회 + bcrypt 비교로 교체
        if (parsed.data.email === 'admin@example.com' && parsed.data.password === 'password') {
          return { id: '1', name: 'Admin', email: 'admin@example.com', role: 'admin' }
        }
        return null
      },
    }),
  ],
  session: { strategy: 'jwt' },
  pages: { signIn: '/login' },
  callbacks: {
    jwt({ token, user }) {
      if (user) {
        token.id = user.id
        token.role = (user as { role?: string }).role ?? 'user'
      }
      return token
    },
    session({ session, token }) {
      if (token.id) session.user.id = token.id as string
      if (token.role) session.user.role = token.role as string
      return session
    },
  },
})
```

---

## src/middleware.ts

```typescript
export { auth as middleware } from '@/lib/auth'

export const config = {
  matcher: ['/((?!api/auth|_next/static|_next/image|favicon.ico|login).*)'],
}
```

---

## src/app/layout.tsx (Root Layout)

```typescript
import type { Metadata } from 'next'
import { Geist } from 'next/font/google'
import './globals.css'
import { Providers } from '@/components/providers'
import { cn } from '@/lib/utils'

const geist = Geist({ subsets: ['latin'], variable: '--font-geist-sans' })

export const metadata: Metadata = {
  title: { template: '%s | My App', default: 'My App' },
  description: '서비스 설명',
}

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="ko" suppressHydrationWarning className={cn('font-sans', geist.variable)}>
      <body className={geist.className}>
        <Providers>{children}</Providers>
      </body>
    </html>
  )
}
```

> **⚠️ 폰트**: shadcn nova 프리셋은 Geist 폰트가 기본이다.
> `npx shadcn@latest init` 실행 시 Inter와 Geist가 혼재될 수 있으니 Geist만 남긴다.
> font variable은 `--font-geist-sans`로 설정하고, `globals.css`의 `@theme inline`에서
> `--font-sans: var(--font-geist-sans, ...)` 로 참조한다.

---

## src/components/providers.tsx ('use client')

```typescript
'use client'

import { SessionProvider } from 'next-auth/react'
import { ThemeProvider } from 'next-themes'
import { Toaster } from '@/components/ui/sonner'

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <ThemeProvider attribute="class" defaultTheme="system" enableSystem>
      <SessionProvider>
        {children}
        <Toaster />
      </SessionProvider>
    </ThemeProvider>
  )
}
```

> **⚠️ 중요**:
> - `Toaster`는 `sonner` 패키지 기반 `@/components/ui/sonner` 사용
>   (`shadcn add sonner` 로 생성 — `toast`는 nova 프리셋에 없음)
> - `ThemeProvider`는 `next-themes` 패키지, shadcn `sonner.tsx`가 `useTheme()` 사용하므로 필수
> - TanStack Query가 필요한 프로젝트는 `QueryClientProvider`를 추가한다

---

## .env.example

```
# Auth.js / NextAuth
NEXTAUTH_SECRET=your-secret-here
NEXTAUTH_URL=http://localhost:3000

# Database
DATABASE_URL=postgresql://user:password@localhost:5432/mydb

# Admin 초기 계정 (seed 스크립트 전용)
ADMIN_EMAIL=admin@example.com
ADMIN_PASSWORD=change-me-in-production
```

---

## shadcn/ui 초기화 순서

```bash
# 1. 의존성 설치
npm install --legacy-peer-deps

# 2. shadcn 초기화 (nova 프리셋 선택)
npx shadcn@latest init
# → 프리셋 선택: Nova (Geist 폰트 + Lucide 아이콘)
# → utils.ts 등 덮어쓰기 발생하므로 init 후 커스텀 헬퍼 재추가 필요

# 3. 컴포넌트 추가 (toast → sonner 사용, toast는 nova에 없음)
npx shadcn@latest add button input label card badge progress \
  accordion dialog alert-dialog select sonner tabs radio-group separator

# 4. providers.tsx에 ThemeProvider 추가 (sonner.tsx가 useTheme() 의존)
# → template의 providers.tsx 참조
```

---

## dotenv 로드 주의사항

Next.js의 `.env.local`은 Next.js 런타임이 자동으로 로드하지만,
`tsx`로 직접 실행하는 스크립트(seed, migration 등)나 `drizzle-kit` CLI는 로드하지 않는다.

```typescript
// ✅ 올바른 방법 — seed.ts, drizzle.config.ts 등 CLI 스크립트 최상단에
import { config } from 'dotenv'
config({ path: '.env.local' })

// 이후 import ...

// ❌ 잘못된 방법 — .env 만 로드하고 .env.local은 무시됨
import 'dotenv/config'
```

> `dotenv`는 devDependency로 설치: `npm install dotenv --save-dev --legacy-peer-deps`
