---
name: ai-dlc-vue-query-guide
description: AI-DLC 개발단계(프론트엔드-Vue.js) 가이드 스킬. @tanstack/vue-query 서버 상태 관리 사용법을 안내한다. "Vue Query 사용법", "useQuery 설정", "vue-query 데이터 패칭", "useMutation Vue", "invalidateQueries Vue", "Vue Query DevTools" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC @tanstack/vue-query 가이드

Vue.js에서 서버 상태(API 데이터)를 관리하는 @tanstack/vue-query 사용법을 안내한다.

## 트리거

- "Vue Query 사용법", "useQuery 설정", "vue-query 데이터 패칭"
- "useMutation Vue", "invalidateQueries Vue", "Vue Query DevTools"
- "Vue TanStack Query 가이드", "@tanstack/vue-query 사용법"

---

## 설치 및 설정

```bash
npm install @tanstack/vue-query @tanstack/vue-query-devtools
```

```typescript
// src/main.ts
import { VueQueryPlugin, QueryClient } from '@tanstack/vue-query'

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000,      // 5분 동안 fresh 상태 유지
      gcTime: 10 * 60 * 1000,        // 10분 후 가비지 컬렉션
      retry: 1,                       // 실패 시 1회 재시도
      refetchOnWindowFocus: false,    // 창 포커스 시 재요청 비활성
    },
  },
})

app.use(VueQueryPlugin, { queryClient })
```

---

## useQuery (데이터 조회)

```typescript
import { useQuery } from '@tanstack/vue-query'
import { computed, ref } from 'vue'
import { fetchUserList, fetchUserById } from '@/api/user.api'

// 목록 조회
const filters = ref({ page: 1, size: 20 })

const {
  data,          // 응답 데이터
  isLoading,     // 최초 로딩 중
  isFetching,    // 재요청 중 (백그라운드)
  isError,       // 오류 발생
  error,         // 오류 객체
  refetch,       // 수동 재요청
} = useQuery({
  queryKey: computed(() => ['users', 'list', filters.value]),  // 반응형 queryKey
  queryFn: () => fetchUserList(filters.value),
  select: (data) => data.content,  // 응답에서 필요한 부분만 추출
})

// 단건 조회 — enabled 조건부 실행
const userId = ref<number | null>(null)

const { data: user } = useQuery({
  queryKey: computed(() => ['users', 'detail', userId.value]),
  queryFn: () => fetchUserById(userId.value!),
  enabled: computed(() => userId.value !== null),
})
```

---

## queryKey 설계 패턴

```typescript
// queryKey는 배열 계층 구조로 설계
const userKeys = {
  all: ['users'] as const,
  lists: () => [...userKeys.all, 'list'] as const,
  list: (filters: UserFilters) => [...userKeys.lists(), filters] as const,
  details: () => [...userKeys.all, 'detail'] as const,
  detail: (id: number) => [...userKeys.details(), id] as const,
}

// 사용 예시
useQuery({ queryKey: userKeys.list(filters.value), queryFn: ... })
useQuery({ queryKey: userKeys.detail(userId.value), queryFn: ... })

// 목록 전체 무효화
queryClient.invalidateQueries({ queryKey: userKeys.lists() })

// 특정 건 무효화
queryClient.invalidateQueries({ queryKey: userKeys.detail(id) })
```

---

## useMutation (데이터 변경)

```typescript
import { useMutation, useQueryClient } from '@tanstack/vue-query'
import { useRouter } from 'vue-router'

const router = useRouter()
const queryClient = useQueryClient()

// 생성
const { mutate: createUser, isPending: isCreating, isError, error } = useMutation({
  mutationFn: (dto: CreateUserDto) => createUserApi(dto),
  onSuccess: (newUser) => {
    // 목록 캐시 무효화 → 자동 재요청
    queryClient.invalidateQueries({ queryKey: ['users', 'list'] })
    router.push('/users')
  },
  onError: (err) => {
    console.error('사용자 생성 실패:', err)
  },
})

// 수정 — Optimistic Update 패턴
const { mutate: updateUser } = useMutation({
  mutationFn: ({ id, dto }: { id: number; dto: UpdateUserDto }) =>
    updateUserApi(id, dto),
  onSuccess: (updatedUser) => {
    // 해당 건 캐시 업데이트
    queryClient.setQueryData(['users', 'detail', updatedUser.id], updatedUser)
    queryClient.invalidateQueries({ queryKey: ['users', 'list'] })
  },
})

// 삭제
const { mutate: deleteUser } = useMutation({
  mutationFn: (id: number) => deleteUserApi(id),
  onSuccess: (_, deletedId) => {
    queryClient.removeQueries({ queryKey: ['users', 'detail', deletedId] })
    queryClient.invalidateQueries({ queryKey: ['users', 'list'] })
    router.push('/users')
  },
})
```

---

## 로딩·에러 상태 처리 패턴

```vue
<template>
  <div>
    <LoadingSpinner v-if="isLoading" />
    <ErrorMessage v-else-if="isError" :message="error?.message" />
    <template v-else>
      <UserTable :users="users" />
    </template>
  </div>
</template>
```

---

## Prefetching

```typescript
import { useQueryClient } from '@tanstack/vue-query'

const queryClient = useQueryClient()

// 호버 시 미리 데이터 요청
async function prefetchUser(id: number) {
  await queryClient.prefetchQuery({
    queryKey: ['users', 'detail', id],
    queryFn: () => fetchUserById(id),
    staleTime: 10_000,  // 10초 동안 fresh
  })
}
```

---

## Vue Query DevTools

```typescript
// src/main.ts (개발 환경만)
import { VueQueryDevtools } from '@tanstack/vue-query-devtools'
```

```vue
<!-- src/App.vue -->
<template>
  <RouterView />
  <VueQueryDevtools v-if="isDev" />
</template>

<script setup lang="ts">
const isDev = import.meta.env.DEV
</script>
```

---

## Vue Query vs Pinia 구분

| 데이터 | 관리 도구 | 이유 |
|:---|:---|:---|
| API 응답 (서버 상태) | Vue Query | 자동 캐싱·재요청·동기화 |
| 인증 토큰·사용자 정보 | Pinia + 영속성 | 앱 전역 + localStorage 저장 필요 |
| UI 상태 (사이드바 등) | Pinia | 서버와 무관한 클라이언트 상태 |
| 폼 로컬 상태 | ref/reactive | 컴포넌트 내부 임시 상태 |
