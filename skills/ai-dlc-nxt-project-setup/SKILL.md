---
name: ai-dlc-nxt-project-setup
description: AI-DLC 개발단계(프론트엔드-Next.js) 스킬. Next.js 15 App Router 프로젝트 초기 설정 파일을 생성한다. "Next.js 프로젝트 만들어줘", "Next.js 앱 초기화", "create-next-app 설정", "App Router 프로젝트 생성", "Next.js 설정해줘", "넥스트 프로젝트 시작" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Next.js 15 App Router 프로젝트 초기 설정

Next.js 15 App Router 기반 프로젝트의 설정 파일과 초기 디렉터리 구조를 생성한다. `template.md`의 코드를 실제 파일로 작성한다.

## 트리거

- "Next.js 프로젝트 만들어줘", "Next.js 앱 초기화", "create-next-app 설정"
- "App Router 프로젝트 생성", "Next.js 설정해줘", "넥스트 프로젝트 시작"

---

## 입력

### 필수
- 프로젝트명 (디렉터리명)

### 선택
- API 서버 URL (없으면 Route Handlers만 사용)
- 인증 필요 여부 (기본: Auth.js 포함)
- DB 사용 여부 (기본: Prisma 포함)

---

## 생성 절차

1. `Glob`으로 현재 디렉터리 구조 확인
2. `template.md` 참조하여 설정 파일 순서대로 생성:
   - `.npmrc` (`legacy-peer-deps=true` — React 19 peer dep 충돌 방지)
   - `package.json` (의존성 전체)
   - `next.config.ts`
   - `tsconfig.json` (strict mode)
   - `.eslintrc.cjs`
   - `.prettierrc`
   - `postcss.config.js` (**Tailwind v4**: `@tailwindcss/postcss` 플러그인)
   - `tailwind.config.ts` (v4: darkMode + content만, theme.extend 불필요)
   - `src/app/globals.css` (**Tailwind v4**: `@import "tailwindcss"` + `@theme inline`)
   - `src/lib/utils.ts` (cn, formatDate, formatDateTime)
   - `src/lib/auth.ts` (Auth.js 설정)
   - `src/middleware.ts` (인증 미들웨어)
   - `src/app/layout.tsx` (**Geist** 폰트, `--font-geist-sans` variable)
   - `src/components/providers.tsx` (**ThemeProvider** + **sonner** Toaster)
   - `.env.example`
3. **npm install 전** `.npmrc` 생성 확인
4. **shadcn init 후** utils.ts 커스텀 헬퍼 재추가 (init이 덮어씀)
5. **shadcn add 시** `toast` 대신 `sonner` 사용
3. `src/` 디렉터리 구조 생성:
   ```
   src/
   ├── app/
   │   ├── layout.tsx          (Root Layout)
   │   ├── page.tsx            (홈 페이지)
   │   ├── globals.css
   │   ├── (auth)/
   │   │   └── login/page.tsx
   │   └── (dashboard)/
   │       └── layout.tsx
   ├── components/
   │   └── ui/                 (shadcn/ui 컴포넌트 위치)
   ├── lib/
   │   ├── auth.ts
   │   └── utils.ts
   ├── types/
   ├── actions/                (Server Actions)
   └── middleware.ts
   ```

---

## 기술 스택

| 카테고리 | 라이브러리 | 버전 | 비고 |
|:---|:---|:---|:---|
| 프레임워크 | Next.js | 15.x (App Router) | Turbopack 기본 |
| 언어 | TypeScript | 5.x (strict) | |
| 스타일 | Tailwind CSS | **4.x** | `@tailwindcss/postcss` 사용 |
| UI 컴포넌트 | shadcn/ui | 4.8.x+ | nova 프리셋, `sonner` 기반 |
| 아이콘 | lucide-react | **1.x+** | v0.x는 React 19 미호환 |
| 토스트 | sonner | 2.x | toast 대신 sonner 사용 |
| 테마 | next-themes | 0.4.x | ThemeProvider 필수 |
| 인증 | Auth.js (NextAuth.js v5) | 5.x | Credentials Provider |
| ORM | Drizzle ORM 또는 Prisma | latest | 프로젝트별 선택 |
| 폼 검증 | Zod + React Hook Form | latest | |
| 데이터 패칭 | RSC fetch() 우선 + TanStack Query (CC) | - | |

---

## 코딩 원칙

- **RSC 기본**: 기본이 Server Component, 인터랙션 필요 시만 `'use client'`
- **Server Actions**: 폼·뮤테이션은 `actions/` 폴더의 Server Actions 사용
- **`next/image`**: `<img>` 태그 대신 `next/image` 필수
- **`next/link`**: `<a href>` 대신 `next/link` 필수
- **환경 변수**: 클라이언트 노출 변수는 `NEXT_PUBLIC_` 접두사 필수

---

## 산출물

- 프로젝트 루트 설정 파일 일체
- `src/` 초기 디렉터리 구조
