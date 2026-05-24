---
name: ai-dlc-vue-router-guide
description: AI-DLC 개발단계(프론트엔드-Vue.js) 가이드 스킬. Vue Router v4 라우팅 설정·네비게이션 가드·동적 라우트를 안내한다. "Vue Router 사용법", "Vue 라우팅 설정", "네비게이션 가드", "동적 라우트", "Vue Router beforeEach", "라우트 메타", "중첩 라우트" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Vue Router v4 가이드

Vue Router v4 설정·동적 라우트·네비게이션 가드·지연 로딩을 안내한다.

## 트리거

- "Vue Router 사용법", "Vue 라우팅 설정", "네비게이션 가드"
- "동적 라우트", "Vue Router beforeEach", "라우트 메타"
- "중첩 라우트", "Vue Router 설정법", "Vue 라우터 가이드"

---

## 기본 설정

```typescript
// src/router/index.ts
import { createRouter, createWebHistory, type RouteRecordRaw } from 'vue-router'

const routes: RouteRecordRaw[] = [
  {
    path: '/login',
    name: 'login',
    component: () => import('@/views/LoginView.vue'),
    meta: { requiresAuth: false, title: '로그인' },
  },
  {
    path: '/',
    component: () => import('@/components/AppLayout.vue'),
    meta: { requiresAuth: true },
    children: [
      {
        path: '',
        name: 'home',
        component: () => import('@/views/HomeView.vue'),
        meta: { title: '홈' },
      },
      {
        path: 'users',
        children: [
          {
            path: '',
            name: 'users',
            component: () => import('@/views/UsersView.vue'),
            meta: { title: '사용자 목록' },
          },
          {
            path: ':id',
            name: 'user-detail',
            component: () => import('@/views/UserDetailView.vue'),
            meta: { title: '사용자 상세' },
          },
        ],
      },
    ],
  },
]

const router = createRouter({
  history: createWebHistory(import.meta.env.BASE_URL),
  routes,
  scrollBehavior(to, from, savedPosition) {
    if (savedPosition) return savedPosition
    return { top: 0 }
  },
})

export default router
```

---

## useRouter / useRoute

```vue
<script setup lang="ts">
import { useRouter, useRoute } from 'vue-router'

const router = useRouter()
const route = useRoute()

// 현재 경로 정보
const currentPath = route.path         // '/users/123'
const userId = route.params.id          // '123' (string)
const queryKeyword = route.query.q      // '검색어'

// 프로그래밍 방식 네비게이션
function goToUser(id: number) {
  router.push({ name: 'user-detail', params: { id } })
}

function goBack() {
  router.back()
}

function replaceCurrentRoute() {
  // 히스토리 스택에 추가하지 않고 교체
  router.replace('/home')
}
</script>
```

---

## 네비게이션 가드

### 전역 가드 (beforeEach)

```typescript
// src/router/index.ts
import { useAuthStore } from '@/stores/auth'

router.beforeEach(async (to, from) => {
  const authStore = useAuthStore()

  // 인증 필요 페이지인데 미인증
  if (to.meta.requiresAuth && !authStore.isAuthenticated) {
    return {
      name: 'login',
      query: { redirect: to.fullPath },  // 로그인 후 원래 페이지로 돌아가기
    }
  }

  // 이미 로그인된 상태에서 로그인 페이지 접근
  if (to.name === 'login' && authStore.isAuthenticated) {
    return { name: 'home' }
  }

  // 페이지 제목 설정
  if (to.meta.title) {
    document.title = `${to.meta.title} | My App`
  }
})
```

### 라우트 메타 타입 정의

```typescript
// src/types/router.d.ts
import 'vue-router'

declare module 'vue-router' {
  interface RouteMeta {
    requiresAuth?: boolean
    title?: string
    roles?: Array<'ADMIN' | 'USER'>  // 권한 제어 시
  }
}
```

### 컴포넌트 내 가드 (onBeforeRouteLeave)

```vue
<script setup lang="ts">
import { onBeforeRouteLeave } from 'vue-router'

onBeforeRouteLeave((to, from, next) => {
  if (hasUnsavedChanges.value) {
    const confirmed = confirm('저장하지 않은 변경사항이 있습니다. 이동하시겠습니까?')
    next(confirmed)
  } else {
    next()
  }
})
</script>
```

---

## 동적 라우트 파라미터

```vue
<script setup lang="ts">
import { computed, watch } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()

// 숫자 변환 (params는 항상 string)
const userId = computed(() => Number(route.params.id))

// 파라미터 변경 감지 (같은 컴포넌트에서 :id만 변경되는 경우)
watch(
  () => route.params.id,
  (newId) => {
    // 데이터 재조회 등
  },
)
</script>
```

---

## 지연 로딩 (Lazy Loading)

```typescript
// 모든 라우트에 지연 로딩 적용 (Vite 자동 코드 분리)
const routes = [
  {
    path: '/users',
    component: () => import('@/views/UsersView.vue'),  // 동적 import
  },
]

// 청크 명명 (번들 분석 시 식별 용이)
{
  path: '/admin',
  component: () => import(/* webpackChunkName: "admin" */ '@/views/AdminView.vue'),
}
```

---

## RouterLink

```vue
<template>
  <!-- 기본 링크 -->
  <RouterLink to="/users">사용자 목록</RouterLink>

  <!-- 이름 기반 링크 -->
  <RouterLink :to="{ name: 'user-detail', params: { id: user.id } }">
    {{ user.name }}
  </RouterLink>

  <!-- 활성 클래스 커스텀 -->
  <RouterLink
    to="/users"
    active-class="text-primary font-bold"
    exact-active-class="text-primary font-bold underline"
  >
    사용자
  </RouterLink>
</template>
```

---

## 중첩 라우트 (AppLayout)

```vue
<!-- src/components/AppLayout.vue -->
<template>
  <div class="flex h-screen">
    <Sidebar />
    <main class="flex-1 overflow-auto">
      <RouterView />  <!-- 중첩 라우트 렌더링 위치 -->
    </main>
  </div>
</template>
```
