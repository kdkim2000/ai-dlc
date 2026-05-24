---
name: ai-dlc-fe-react-query-guide
description: AI-DLC 개발단계(프론트엔드-React) 스킬. TanStack Query(React Query) 서버 상태 관리 가이드를 제공한다. "React Query 가이드", "TanStack Query 사용법", "서버 상태 관리", "데이터 패칭 방법", "useQuery 어떻게 써", "useMutation 사용법", "queryKey 설계" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC TanStack Query (React Query) 서버 상태 관리 가이드

QueryClient 설정·useQuery·useMutation·queryKey 설계·무한 스크롤·Axios 연동 패턴을 대화창에 출력한다. 파일을 수정하지 않는다.

## 트리거

- "React Query 가이드", "TanStack Query 사용법", "서버 상태 관리"
- "데이터 패칭 방법", "useQuery 어떻게 써", "useMutation 사용법"
- "queryKey 설계", "invalidateQueries 사용법", "캐시 무효화"

---

## 초기 설정 (`src/lib/queryClient.ts`)

```typescript
import { QueryClient } from '@tanstack/react-query'

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,    // 5분: 캐시된 데이터를 신선하게 간주
      gcTime: 1000 * 60 * 10,      // 10분: 미사용 캐시 메모리 보존 시간
      retry: 1,
      refetchOnWindowFocus: false,  // 탭 전환 시 자동 재요청 비활성화
    },
  },
})

// src/main.tsx
import { QueryClientProvider } from '@tanstack/react-query'
import { ReactQueryDevtools } from '@tanstack/react-query-devtools'
import { queryClient } from '@/lib/queryClient'

ReactDOM.createRoot(document.getElementById('root')!).render(
  <QueryClientProvider client={queryClient}>
    <App />
    {import.meta.env.DEV && <ReactQueryDevtools initialIsOpen={false} />}
  </QueryClientProvider>
)
```

---

## queryKey 설계 원칙 (`src/api/queryKeys.ts`)

```typescript
// 계층적 구조로 설계 — 상위 키 무효화 시 하위 키 모두 무효화
export const userKeys = {
  all: ['users'] as const,
  list: (filters: UserFilter) => ['users', 'list', filters] as const,
  detail: (userId: number) => ['users', 'detail', userId] as const,
}

// 사용 예시
// userKeys.all         → ['users']
// userKeys.list({})    → ['users', 'list', {}]
// userKeys.detail(1)   → ['users', 'detail', 1]

// invalidateQueries 범위:
// queryClient.invalidateQueries({ queryKey: userKeys.all })    → users 관련 전체 무효화
// queryClient.invalidateQueries({ queryKey: ['users', 'list'] }) → 목록만 무효화
```

---

## useQuery — 데이터 조회

```typescript
// src/hooks/useUserList.ts
import { useQuery } from '@tanstack/react-query'
import { userApi } from '@/api/user.api'
import { userKeys } from '@/api/queryKeys'

export function useUserList(filters: UserFilter) {
  return useQuery({
    queryKey: userKeys.list(filters),
    queryFn: ({ signal }) => userApi.getList(filters, signal),  // AbortController 자동 연동
    staleTime: 1000 * 60 * 5,
    placeholderData: (prev) => prev,  // 필터 변경 시 이전 데이터 유지 (페이지 깜박임 방지)
  })
}

// 컴포넌트에서 사용
function UserListPage() {
  const [filters, setFilters] = useState<UserFilter>({ page: 0, size: 20 })
  const { data, isLoading, isError, error } = useUserList(filters)

  if (isLoading) return <Skeleton />
  if (isError) return <ErrorMessage error={error} />

  return <DataTable data={data?.content ?? []} />
}
```

---

## useQuery 조건부 실행

```typescript
// enabled 옵션으로 조건부 실행
export function useUserDetail(userId: number | null) {
  return useQuery({
    queryKey: userKeys.detail(userId!),
    queryFn: () => userApi.getOne(userId!),
    enabled: userId !== null,  // userId가 null이면 쿼리 실행 안 함
  })
}
```

---

## useMutation — 데이터 변경

```typescript
// src/hooks/useCreateUser.ts
import { useMutation, useQueryClient } from '@tanstack/react-query'
import { userApi } from '@/api/user.api'
import { userKeys } from '@/api/queryKeys'
import { useToast } from '@/hooks/use-toast'
import { getErrorMessage } from '@/lib/axios'

export function useCreateUser() {
  const queryClient = useQueryClient()
  const { toast } = useToast()

  return useMutation({
    mutationFn: userApi.create,
    onSuccess: (newUser) => {
      // 1. 목록 캐시 무효화 (자동 재조회)
      queryClient.invalidateQueries({ queryKey: userKeys.all })
      toast({ title: '등록 완료', description: `${newUser.userNm} 등록되었습니다.` })
    },
    onError: (error) => {
      toast({ title: '등록 실패', description: getErrorMessage(error), variant: 'destructive' })
    },
  })
}

// 컴포넌트에서 사용
function UserCreatePage() {
  const { mutate: createUser, isPending } = useCreateUser()
  const navigate = useNavigate()

  const onSubmit = (data: UserCreateReq) => {
    createUser(data, {
      onSuccess: () => navigate('/users'),
    })
  }
}
```

---

## Optimistic Update (낙관적 업데이트)

```typescript
export function useUpdateUser() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: ({ userId, body }: { userId: number; body: UserUpdateReq }) =>
      userApi.update(userId, body),

    // UI를 API 응답 전에 즉시 업데이트
    onMutate: async ({ userId, body }) => {
      await queryClient.cancelQueries({ queryKey: userKeys.detail(userId) })
      const previousUser = queryClient.getQueryData<UserVO>(userKeys.detail(userId))

      queryClient.setQueryData(userKeys.detail(userId), (old: UserVO | undefined) =>
        old ? { ...old, ...body } : old
      )

      return { previousUser }  // 롤백 데이터
    },

    onError: (_err, { userId }, context) => {
      // 에러 시 이전 데이터로 롤백
      queryClient.setQueryData(userKeys.detail(userId), context?.previousUser)
    },

    onSettled: (_data, _err, { userId }) => {
      queryClient.invalidateQueries({ queryKey: userKeys.detail(userId) })
    },
  })
}
```

---

## 무한 스크롤 (useInfiniteQuery)

```typescript
export function useInfiniteUserList(filters: Omit<UserFilter, 'page'>) {
  return useInfiniteQuery({
    queryKey: userKeys.list({ ...filters, infinite: true }),
    queryFn: ({ pageParam = 0 }) => userApi.getList({ ...filters, page: pageParam }),
    getNextPageParam: (lastPage) =>
      lastPage.page + 1 < lastPage.totalPages ? lastPage.page + 1 : undefined,
    initialPageParam: 0,
  })
}

// 컴포넌트
const { data, fetchNextPage, hasNextPage, isFetchingNextPage } = useInfiniteUserList({})

const allUsers = data?.pages.flatMap((page) => page.content) ?? []
```

---

## 서버 상태 prefetch (SSR / 초기 로드 최적화)

```typescript
// 라우터 loader에서 미리 캐시 채우기 (React Router v6.4+ loader)
export async function userListLoader() {
  await queryClient.prefetchQuery({
    queryKey: userKeys.list({ page: 0, size: 20 }),
    queryFn: () => userApi.getList({ page: 0, size: 20 }),
  })
  return null
}
```

---

## suspense 모드

```typescript
// React 18 Suspense와 연동
const { data } = useSuspenseQuery({
  queryKey: userKeys.list(filters),
  queryFn: () => userApi.getList(filters),
})

// 사용 — Suspense + ErrorBoundary 필수
<ErrorBoundary fallback={<ErrorPage />}>
  <Suspense fallback={<Skeleton />}>
    <UserList />
  </Suspense>
</ErrorBoundary>
```
