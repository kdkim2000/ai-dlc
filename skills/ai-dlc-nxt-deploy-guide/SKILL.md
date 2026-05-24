---
name: ai-dlc-nxt-deploy-guide
description: AI-DLC 개발단계(프론트엔드-Next.js) 스킬. Next.js 배포 가이드를 제공한다. "Next.js 배포 방법", "Vercel 배포", "Next.js Docker", "standalone 빌드", "env vars 배포", "ISR 전략" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Next.js 배포 가이드

## 트리거

- "Next.js 배포 방법", "Vercel 배포"
- "Next.js Docker", "standalone 빌드"
- "env vars 배포", "ISR 전략"

---

## 1. Vercel 배포 (권장)

### 프로젝트 연결
```bash
npm install -g vercel
vercel login
vercel --prod
```

### 환경 변수 설정
```bash
# Vercel CLI
vercel env add AUTH_SECRET production
vercel env add DATABASE_URL production

# 또는 Vercel 대시보드 → Settings → Environment Variables
```

### Preview 브랜치
- `main` → Production
- 그 외 브랜치 → Preview URL 자동 생성 (`https://[branch].vercel.app`)
- PR마다 Preview 댓글 자동 등록

---

## 2. Standalone Docker 빌드

### next.config.ts 설정
```typescript
const nextConfig: NextConfig = {
  output: 'standalone',  // standalone 빌드 활성화
}
```

### Dockerfile
```dockerfile
FROM node:20-alpine AS base

# 의존성 설치
FROM base AS deps
WORKDIR /app
COPY package.json package-lock.json ./
RUN npm ci

# 빌드
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ENV NEXT_TELEMETRY_DISABLED 1
RUN npm run build

# 실행
FROM base AS runner
WORKDIR /app
ENV NODE_ENV production
ENV NEXT_TELEMETRY_DISABLED 1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs
EXPOSE 3000
ENV PORT 3000
ENV HOSTNAME "0.0.0.0"

CMD ["node", "server.js"]
```

### docker-compose.yml
```yaml
version: '3.8'
services:
  web:
    build: .
    ports:
      - '3000:3000'
    environment:
      - AUTH_SECRET=${AUTH_SECRET}
      - DATABASE_URL=${DATABASE_URL}
      - API_BASE_URL=${API_BASE_URL}
    restart: unless-stopped
```

---

## 3. 환경 변수 전략

| 변수명 | 접근 위치 | 예시 |
|:---|:---|:---|
| `AUTH_SECRET` | 서버 전용 | Auth.js 서명 키 |
| `DATABASE_URL` | 서버 전용 | PostgreSQL 연결 문자열 |
| `API_BASE_URL` | 서버 전용 | 외부 API 베이스 URL |
| `NEXT_PUBLIC_APP_URL` | 클라이언트+서버 | 앱 공개 URL |
| `NEXT_PUBLIC_GA_ID` | 클라이언트+서버 | Google Analytics ID |

**규칙**: 클라이언트에서 접근해야 하는 변수만 `NEXT_PUBLIC_` 접두사 사용. 비밀값에 `NEXT_PUBLIC_` 금지.

---

## 4. 헬스체크 엔드포인트

```typescript
// app/api/health/route.ts
import { NextResponse } from 'next/server'

export async function GET() {
  try {
    // DB 연결 확인 (선택)
    // await prisma.$queryRaw`SELECT 1`
    return NextResponse.json({ status: 'ok', timestamp: new Date().toISOString() })
  } catch {
    return NextResponse.json({ status: 'error' }, { status: 503 })
  }
}
```

---

## 5. ISR 전략별 적용 가이드

| 페이지 유형 | 전략 | revalidate |
|:---|:---|:---|
| 마케팅·정적 페이지 | SSG | `force-cache` |
| 상품 목록 | ISR | `revalidate: 300` (5분) |
| 사용자 목록 | ISR | `revalidate: 60` (1분) |
| 실시간 재고·가격 | SSR | `no-store` |
| 사용자 대시보드 | SSR | `no-store` |

### 태그 기반 무효화 (On-demand ISR)
```typescript
// fetch 시 태그 설정
fetch(url, { next: { tags: ['products', `product-${id}`] } })

// Server Action 또는 Webhook에서 무효화
import { revalidateTag } from 'next/cache'
revalidateTag('products')        // 모든 products 캐시 무효화
revalidateTag(`product-${id}`)   // 특정 상품만 무효화
```

---

## 6. CI/CD (GitHub Actions 예시)

```yaml
# .github/workflows/deploy.yml
name: Deploy to Vercel

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
          vercel-args: '--prod'
```
