---
name: ai-dlc-nxt-perf-guide
description: AI-DLC 개발단계(프론트엔드-Next.js) 스킬. Next.js 성능 최적화 가이드를 제공한다. "Next.js 성능 최적화", "next/image 사용법", "Next.js 빌드 최적화", "next/font", "Dynamic Import", "번들 최적화" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Next.js 성능 최적화 가이드

## 트리거

- "Next.js 성능 최적화", "next/image 사용법"
- "Next.js 빌드 최적화", "next/font"
- "Dynamic Import", "번들 최적화"

---

## next/image — 이미지 최적화

```typescript
import Image from 'next/image'

// 고정 크기 이미지
<Image
  src="/hero.jpg"
  alt="히어로 이미지"
  width={1200}
  height={600}
  priority           // LCP 이미지: 즉시 로드
  placeholder="blur" // blurDataURL 필요 시
/>

// 반응형 이미지 (fill 모드)
<div className="relative h-64 w-full">
  <Image
    src={user.avatar}
    alt={user.name}
    fill
    sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
    className="object-cover"
  />
</div>

// 외부 이미지 — next.config.ts에 도메인 등록 필수
// images: { remotePatterns: [{ hostname: 'cdn.example.com' }] }
```

---

## next/font — 폰트 최적화

```typescript
// app/layout.tsx
import { Inter, Noto_Sans_KR } from 'next/font/google'

const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter',
})

const notoSansKr = Noto_Sans_KR({
  subsets: ['latin'],
  weight: ['400', '500', '700'],
  display: 'swap',
  variable: '--font-noto-sans-kr',
})

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html className={`${inter.variable} ${notoSansKr.variable}`}>
      <body>{children}</body>
    </html>
  )
}
```

---

## Dynamic Import — 코드 분할

```typescript
import dynamic from 'next/dynamic'

// 클라이언트 전용 컴포넌트 (SSR 비활성화)
const Chart = dynamic(() => import('@/components/Chart'), {
  ssr: false,
  loading: () => <ChartSkeleton />,
})

// 조건부 로드 (모달, 드로어 등)
const HeavyModal = dynamic(() => import('@/components/HeavyModal'), {
  loading: () => null,
})

// 사용
function Page() {
  const [showModal, setShowModal] = useState(false)
  return (
    <>
      <button onClick={() => setShowModal(true)}>열기</button>
      {showModal && <HeavyModal />}
    </>
  )
}
```

---

## Metadata API — SEO 최적화

```typescript
// 정적 메타데이터
export const metadata: Metadata = {
  title: { template: '%s | My App', default: 'My App' },
  description: '서비스 설명',
  openGraph: {
    title: 'My App',
    description: '서비스 설명',
    images: [{ url: '/og-image.png', width: 1200, height: 630 }],
  },
}

// 동적 메타데이터 (RSC page.tsx)
export async function generateMetadata({ params }: Props): Promise<Metadata> {
  const data = await getUser(params.id)
  return {
    title: data.name,
    description: `${data.name}의 프로필 페이지`,
  }
}
```

---

## Bundle Analyzer — 번들 크기 분석

```bash
# 설치
npm install @next/bundle-analyzer

# next.config.ts
import withBundleAnalyzer from '@next/bundle-analyzer'

const bundleAnalyzer = withBundleAnalyzer({
  enabled: process.env.ANALYZE === 'true',
})

export default bundleAnalyzer(nextConfig)

# 실행
ANALYZE=true npm run build
```

---

## Route Prefetching

```typescript
// Link 컴포넌트: viewport에 들어오면 자동 prefetch (기본값)
<Link href="/users" prefetch={false}>사용자 목록</Link>
// prefetch={false}: 대역폭 절약이 필요한 경우

// Router.prefetch: 클릭 전 미리 fetch
import { useRouter } from 'next/navigation'
const router = useRouter()
<button onMouseEnter={() => router.prefetch('/users/new')}>사용자 등록</button>
```

---

## 렌더링 전략 요약

| 전략 | 설정 | 특징 |
|:---|:---|:---|
| SSG (정적) | `cache: 'force-cache'` | 빌드 시 생성, CDN 캐시 |
| ISR (증분 재생성) | `revalidate: N` | N초 후 백그라운드 갱신 |
| SSR (서버 렌더링) | `cache: 'no-store'` | 매 요청마다 서버 실행 |
| CSR (클라이언트) | `'use client'` + TanStack Query | 클라이언트 fetch |

**권장**: 목록 ISR(60s) → 상세 force-cache → 실시간 no-store → 검색 CSR
