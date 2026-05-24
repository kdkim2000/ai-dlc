---
name: ai-dlc-fe-perf-guide
description: AI-DLC 개발단계(프론트엔드-React) 스킬. React 성능 최적화 가이드를 제공한다. "React 성능 최적화", "리렌더링 방지", "React 최적화 방법", "useMemo useCallback 언제 써", "React.memo 사용법", "코드 스플리팅", "번들 최적화" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC React 성능 최적화 가이드

React 리렌더링 최적화(memo/useMemo/useCallback)·코드 스플리팅·번들 분석·React Query 캐싱 활용 패턴을 대화창에 출력한다. 파일을 수정하지 않는다.

## 트리거

- "React 성능 최적화", "리렌더링 방지", "React 최적화 방법"
- "useMemo useCallback 언제 써", "React.memo 사용법"
- "코드 스플리팅", "번들 최적화", "lazy loading"

---

## 리렌더링 원인과 확인 방법

```typescript
// React DevTools Profiler로 불필요한 리렌더링 확인
// 또는 임시 디버깅
const MyComponent = () => {
  console.log('렌더링')  // 개발 중에만 사용, 배포 전 제거
  return <div>...</div>
}
```

---

## React.memo — 언제 사용할까

```tsx
// 권장: Props가 자주 바뀌지 않는 순수 컴포넌트
const UserCard = React.memo(function UserCard({ user }: { user: UserVO }) {
  return (
    <div>
      <p>{user.userNm}</p>
      <p>{user.email}</p>
    </div>
  )
})

// 불필요: 부모가 리렌더될 때 항상 Props가 달라지는 경우 효과 없음
// 불필요: 내부에 무거운 연산이 없는 단순 컴포넌트

// 커스텀 비교 (특정 필드만 비교)
const UserCard = React.memo(UserCardBase, (prev, next) =>
  prev.user.userId === next.user.userId
)
```

---

## useMemo — 무거운 계산 메모이제이션

```typescript
// 권장: 배열 정렬·필터 같은 O(n) 이상 연산
const sortedUsers = useMemo(
  () => [...users].sort((a, b) => a.userNm.localeCompare(b.userNm)),
  [users]  // users 참조가 바뀔 때만 재계산
)

// 권장: 파생 상태 계산
const activeCount = useMemo(
  () => users.filter((u) => u.useYn === 'Y').length,
  [users]
)

// 불필요: 단순 값 변환 (오히려 오버헤드)
const userName = useMemo(() => user.userNm.toUpperCase(), [user.userNm])
// → const userName = user.userNm.toUpperCase()  로 직접 작성
```

---

## useCallback — 함수 참조 안정화

```typescript
// 권장: React.memo 자식에게 콜백 전달할 때
const handleDelete = useCallback((userId: number) => {
  deleteUser(userId)
}, [deleteUser])  // deleteUser가 useMutation의 안정적 참조일 때

// 권장: useEffect 의존성에 함수가 포함될 때
const fetchData = useCallback(async () => {
  const data = await userApi.getList(filters)
  setUsers(data)
}, [filters])

useEffect(() => {
  fetchData()
}, [fetchData])

// 불필요: 이벤트 핸들러를 React.memo가 아닌 자식에게 전달하는 경우
```

---

## useEffect 의존성 배열 올바른 사용

```typescript
// 위반 — 의존성 누락
useEffect(() => {
  fetchUser(userId)
}, [])

// 위반 — 객체 의존성 (매 렌더마다 새 객체 생성)
useEffect(() => {
  fetchUser(filters)
}, [{ userNm, page }])  // 매번 새 객체!

// 권장 — 원시값 의존성
useEffect(() => {
  fetchUser({ userNm, page })
}, [userNm, page])

// 권장 — React Query 사용 (useEffect + fetch 패턴 제거)
const { data } = useQuery({
  queryKey: ['users', userNm, page],
  queryFn: () => userApi.getList({ userNm, page }),
})
```

---

## Code Splitting (React.lazy + Suspense)

```tsx
// src/App.tsx — 라우트 단위 코드 스플리팅
import { lazy, Suspense } from 'react'

const UserListPage = lazy(() => import('@/pages/UserListPage'))
const UserCreatePage = lazy(() => import('@/pages/UserCreatePage'))

function App() {
  return (
    <Suspense fallback={<div className="flex justify-center p-8">로딩 중...</div>}>
      <Routes>
        <Route path="/users" element={<UserListPage />} />
        <Route path="/users/create" element={<UserCreatePage />} />
      </Routes>
    </Suspense>
  )
}
```

---

## 번들 분석 (vite-bundle-visualizer)

```bash
# 설치
npm install --save-dev rollup-plugin-visualizer

# vite.config.ts
import { visualizer } from 'rollup-plugin-visualizer'

export default defineConfig({
  plugins: [
    react(),
    visualizer({ open: true, gzipSize: true, brotliSize: true }),
  ],
})

# 빌드 후 자동으로 브라우저에서 번들 분석 결과 표시
npm run build
```

---

## React Query 캐싱으로 API 중복 호출 방지

```typescript
// QueryClient 설정 최적화
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 1000 * 60 * 5,   // 5분 — 동일 쿼리 재요청 방지
      gcTime: 1000 * 60 * 10,     // 10분 — 메모리에 캐시 유지
      retry: 1,
      refetchOnWindowFocus: false, // 탭 전환 시 재요청 방지 (필요 시 true)
    },
  },
})

// prefetchQuery로 다음 페이지 미리 로드
const prefetchNextPage = () => {
  queryClient.prefetchQuery({
    queryKey: userKeys.list({ ...filters, page: filters.page + 1 }),
    queryFn: () => userApi.getList({ ...filters, page: filters.page + 1 }),
  })
}
```

---

## 대용량 목록 가상화 (react-window)

```tsx
// 1000개 이상 목록에서 DOM 노드 최소화
import { FixedSizeList } from 'react-window'

<FixedSizeList
  height={600}
  itemCount={users.length}
  itemSize={60}
  width="100%"
>
  {({ index, style }) => (
    <div style={style}>
      <UserRow user={users[index]} />
    </div>
  )}
</FixedSizeList>
```

---

## 불필요한 state 피하기

```typescript
// 안티패턴 — 파생 상태를 별도 useState에 저장
const [activeCount, setActiveCount] = useState(0)
useEffect(() => {
  setActiveCount(users.filter((u) => u.useYn === 'Y').length)
}, [users])

// 권장 — useMemo로 파생 계산
const activeCount = useMemo(
  () => users.filter((u) => u.useYn === 'Y').length,
  [users]
)
```
