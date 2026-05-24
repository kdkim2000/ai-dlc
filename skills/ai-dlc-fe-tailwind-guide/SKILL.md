---
name: ai-dlc-fe-tailwind-guide
description: AI-DLC 개발단계(프론트엔드-React) 스킬. Tailwind CSS 활용 가이드를 제공한다. "Tailwind CSS 가이드", "테일윈드 사용법", "CSS 유틸리티 클래스", "반응형 디자인 방법", "Tailwind 반응형", "Tailwind 색상 커스텀", "테일윈드 설정" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Tailwind CSS 활용 가이드

Tailwind CSS 유틸리티 클래스 패턴, 반응형 디자인, 커스텀 설정, cn() 활용법을 대화창에 출력한다. 파일을 수정하지 않는다.

## 트리거

- "Tailwind CSS 가이드", "테일윈드 사용법", "CSS 유틸리티 클래스"
- "반응형 디자인 방법", "Tailwind 반응형", "Tailwind 색상 커스텀"
- "테일윈드 설정", "Tailwind @apply", "cn() 사용법"

---

## 반응형 디자인

```tsx
// 브레이크포인트: sm(640px), md(768px), lg(1024px), xl(1280px), 2xl(1536px)

// 모바일 퍼스트: 기본값이 모바일, 접두사가 해당 너비 이상
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {/* 모바일: 1열, 태블릿: 2열, 데스크탑: 3열 */}
</div>

// 숨기기/보이기
<div className="hidden md:block">태블릿 이상에서만 표시</div>
<div className="md:hidden">모바일에서만 표시</div>

// 패딩/마진 반응형
<div className="p-4 md:p-6 lg:p-8">...</div>
```

---

## 자주 쓰는 레이아웃 패턴

### Flex
```tsx
// 가로 중앙 정렬
<div className="flex items-center justify-between">

// 수직 가운데 정렬 (전체 높이)
<div className="flex flex-col items-center justify-center min-h-screen">

// 간격 포함 flex
<div className="flex items-center gap-2">
  <Icon />
  <span>텍스트</span>
</div>
```

### Grid
```tsx
// 기본 그리드
<div className="grid grid-cols-12 gap-4">
  <div className="col-span-12 md:col-span-8">메인</div>
  <div className="col-span-12 md:col-span-4">사이드</div>
</div>
```

### Card 레이아웃
```tsx
<div className="rounded-lg border bg-card shadow-sm p-6">
  <h3 className="text-lg font-semibold">제목</h3>
  <p className="text-sm text-muted-foreground mt-1">설명</p>
</div>
```

---

## 색상 시스템 (shadcn/ui 연동)

```tsx
// shadcn/ui CSS 변수 기반 색상 — 테마 전환 지원
<div className="bg-background text-foreground">기본 배경</div>
<div className="bg-card text-card-foreground">카드</div>
<div className="bg-primary text-primary-foreground">주색상 배경</div>
<div className="bg-muted text-muted-foreground">음소거 색상</div>
<div className="bg-destructive text-destructive-foreground">위험 액션</div>
<div className="border border-border">테두리</div>
```

---

## Typography

```tsx
<h1 className="text-3xl font-bold tracking-tight">제목 1</h1>
<h2 className="text-2xl font-semibold">제목 2</h2>
<p className="text-sm text-muted-foreground">보조 텍스트</p>
<span className="text-xs font-medium uppercase tracking-wide">레이블</span>

// 텍스트 자르기
<p className="truncate max-w-[200px]">긴 텍스트...</p>
<p className="line-clamp-2">두 줄까지만 표시되는 텍스트</p>
```

---

## 상태별 스타일

```tsx
// 비활성화
<button
  className="opacity-50 cursor-not-allowed"
  disabled
>...</button>

// 호버/포커스
<button className="hover:bg-accent hover:text-accent-foreground focus-visible:ring-2 focus-visible:ring-ring">

// 선택 상태
<div className={cn('border rounded-md p-2', isSelected && 'border-primary bg-primary/10')}>
```

---

## @apply로 반복 클래스 추출

```css
/* src/index.css — 자주 쓰는 패턴 추출 */
@layer components {
  .btn-primary {
    @apply inline-flex items-center justify-center rounded-md bg-primary
           px-4 py-2 text-sm font-medium text-primary-foreground
           hover:bg-primary/90 focus-visible:outline-none focus-visible:ring-2;
  }

  .form-input {
    @apply flex h-10 w-full rounded-md border border-input bg-background
           px-3 py-2 text-sm ring-offset-background
           placeholder:text-muted-foreground
           focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring;
  }

  .page-container {
    @apply container mx-auto px-4 py-6 max-w-7xl;
  }
}
```

> **주의**: shadcn/ui 컴포넌트 파일(`src/components/ui/`)에는 `@apply` 사용 금지 — 컴포넌트 업데이트 시 충돌 발생.

---

## tailwind.config.ts 커스터마이징

```typescript
import type { Config } from 'tailwindcss'

const config: Config = {
  content: ['./index.html', './src/**/*.{ts,tsx}'],
  theme: {
    extend: {
      colors: {
        // 브랜드 색상 추가
        brand: {
          50: '#eff6ff',
          500: '#3b82f6',
          900: '#1e3a5f',
        },
      },
      fontFamily: {
        sans: ['Pretendard', 'sans-serif'],
      },
      spacing: {
        '18': '4.5rem',
        '88': '22rem',
      },
    },
  },
  plugins: [require('tailwindcss-animate')],
}
export default config
```

---

## cn() 유틸리티 패턴

```typescript
import { cn } from '@/utils/cn'

// 조건부 클래스
<div className={cn('base-class', condition && 'conditional-class')}>

// Props로 클래스 받기
interface Props {
  className?: string
}
function MyComponent({ className }: Props) {
  return <div className={cn('default-styles', className)}>
}

// 변형(variant) 패턴
const variants = {
  default: 'bg-primary text-primary-foreground',
  outline: 'border border-input bg-background',
  ghost: 'hover:bg-accent hover:text-accent-foreground',
} as const

<button className={cn('base', variants[variant])}>
```
