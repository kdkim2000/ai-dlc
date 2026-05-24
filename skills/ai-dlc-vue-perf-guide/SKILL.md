---
name: ai-dlc-vue-perf-guide
description: AI-DLC 개발단계(프론트엔드-Vue.js) 가이드 스킬. Vue.js 성능 최적화 기법을 안내한다. "Vue 성능 최적화", "Vue 렌더링 최적화", "Vite 번들 최적화", "defineAsyncComponent", "v-memo", "KeepAlive Vue", "Vue 번들 분석" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Vue.js 성능 최적화 가이드

Vue.js 렌더링 최적화·코드 분리·번들 분석 기법을 안내한다.

## 트리거

- "Vue 성능 최적화", "Vue 렌더링 최적화", "Vite 번들 최적화"
- "defineAsyncComponent", "v-memo", "KeepAlive Vue"
- "Vue 번들 분석", "Vue 메모리 최적화", "Vue shallowRef"

---

## 1. defineAsyncComponent (지연 로딩)

```typescript
// 컴포넌트 지연 로딩
import { defineAsyncComponent } from 'vue'

// 기본 사용
const HeavyChart = defineAsyncComponent(
  () => import('@/components/HeavyChart.vue')
)

// 로딩·에러 컴포넌트 포함
const LazyModal = defineAsyncComponent({
  loader: () => import('@/components/HeavyModal.vue'),
  loadingComponent: LoadingSpinner,
  errorComponent: ErrorMessage,
  delay: 200,       // 200ms 후 로딩 컴포넌트 표시
  timeout: 10000,   // 10초 초과 시 에러
})
```

```vue
<template>
  <!-- 조건부 렌더링 시 지연 로딩 효과적 -->
  <LazyModal v-if="showModal" />
</template>
```

---

## 2. v-memo (조건부 재렌더 방지)

```vue
<template>
  <!-- 의존성 배열이 변경될 때만 재렌더 -->
  <div v-for="item in list" :key="item.id" v-memo="[item.id, item.selected]">
    <HeavyItem :item="item" />
  </div>

  <!-- 정적 섹션 메모화 (의존성 없음) -->
  <div v-memo="[]">
    <StaticHeader />
  </div>
</template>
```

---

## 3. shallowRef vs ref 선택

```typescript
// ref: 깊은 반응성 (중첩 객체의 변경도 감지)
const user = ref<User>({ name: 'Alice', address: { city: 'Seoul' } })
user.value.address.city = 'Busan'  // 반응성 트리거 O

// shallowRef: 얕은 반응성 (최상위 변경만 감지)
const users = shallowRef<User[]>([])
users.value = [...users.value, newUser]  // 반응성 트리거 O
users.value[0].name = 'Bob'  // 반응성 트리거 X — 직접 push 금지

// 사용 기준
// - 대형 배열·객체는 shallowRef 사용 (렌더링 성능 향상)
// - 내부 필드 변경이 필요하면 ref 사용
// - read-only 데이터는 Object.freeze() + shallowRef 조합

const config = shallowRef(Object.freeze({ apiUrl: '...' }))
```

---

## 4. KeepAlive (컴포넌트 캐싱)

```vue
<!-- 탭 전환 시 상태 유지 -->
<template>
  <KeepAlive :include="['UsersView', 'ProductsView']" :max="5">
    <component :is="currentView" />
  </KeepAlive>
</template>

<!-- RouterView와 함께 사용 -->
<template>
  <RouterView v-slot="{ Component, route }">
    <KeepAlive :include="keepAliveRoutes">
      <component :is="Component" :key="route.name" />
    </KeepAlive>
  </RouterView>
</template>
```

```typescript
// 라우트 메타로 캐시 여부 제어
const keepAliveRoutes = computed(() =>
  router.getRoutes()
    .filter(r => r.meta.keepAlive)
    .map(r => r.name as string)
)
```

KeepAlive 컴포넌트 내 lifecycle hooks:
```typescript
import { onActivated, onDeactivated } from 'vue'

onActivated(() => {
  // KeepAlive에서 활성화될 때 (마운트 대신 호출)
  refreshData()
})

onDeactivated(() => {
  // KeepAlive에서 비활성화될 때
})
```

---

## 5. computed 최적화

```typescript
// ❌ template 내 복잡한 표현식
// <div>{{ users.filter(u => u.active).sort((a, b) => a.name.localeCompare(b.name)) }}</div>

// ✅ computed로 캐싱
const sortedActiveUsers = computed(() =>
  users.value
    .filter(u => u.active)
    .sort((a, b) => a.name.localeCompare(b.name))
)

// computed는 의존성이 변경될 때만 재계산
// 반환값이 같으면 자식 컴포넌트 재렌더 방지
```

---

## 6. Vite 번들 분석

```bash
npm install --save-dev rollup-plugin-visualizer
```

```typescript
// vite.config.ts
import { visualizer } from 'rollup-plugin-visualizer'

export default defineConfig({
  plugins: [
    vue(),
    visualizer({
      open: true,          // 빌드 후 자동으로 브라우저 열기
      filename: 'stats.html',
      gzipSize: true,
      brotliSize: true,
    }),
  ],
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor-vue': ['vue', 'vue-router', 'pinia'],
          'vendor-query': ['@tanstack/vue-query'],
          'vendor-form': ['vee-validate', '@vee-validate/zod', 'zod'],
          'vendor-ui': ['radix-vue', 'lucide-vue-next'],
        },
      },
    },
    chunkSizeWarningLimit: 800,  // 800KB 초과 시 경고
  },
})
```

---

## 7. 이미지 최적화

```vue
<template>
  <!-- WebP 형식 + 크기 명시 -->
  <img
    src="/images/hero.webp"
    alt="히어로 이미지"
    width="1200"
    height="600"
    loading="lazy"
    decoding="async"
  />

  <!-- 반응형 이미지 -->
  <img
    srcset="/images/hero-sm.webp 640w, /images/hero.webp 1200w"
    sizes="(max-width: 640px) 640px, 1200px"
    src="/images/hero.webp"
    alt="히어로 이미지"
  />
</template>
```

---

## 성능 체크리스트

| 항목 | 설명 | 적용 여부 |
|:---|:---|:---:|
| `defineAsyncComponent` | 무거운 컴포넌트 지연 로딩 | □ |
| `shallowRef` | 대형 배열·read-only 데이터 | □ |
| `v-memo` | 반복 렌더 비용 높은 목록 | □ |
| `KeepAlive` | 탭·라우트 전환 상태 유지 | □ |
| `computed` 활용 | template 내 복잡 표현식 제거 | □ |
| 번들 분리 | vendor 청크 분리 | □ |
| 이미지 lazy loading | `loading="lazy"` 적용 | □ |
| Vue Query staleTime | 불필요한 API 재요청 감소 | □ |
