---
name: ai-dlc-fe-state-guide
description: AI-DLC 개발단계(프론트엔드-React) 스킬. 상태 관리 가이드(Zustand / Redux Toolkit)를 제공한다. "상태 관리 가이드", "Zustand 사용법", "React 상태 관리", "전역 상태 관리", "Redux 대신 뭐 써", "Zustand store 만들어줘", "전역 상태 어떻게 써" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC 상태 관리 가이드 (Zustand / Redux Toolkit)

클라이언트 상태 관리(Zustand)와 서버 상태 관리(React Query) 구분 원칙, Zustand store 설계 패턴, 미들웨어 활용법을 대화창에 출력한다. 파일을 수정하지 않는다.

## 트리거

- "상태 관리 가이드", "Zustand 사용법", "React 상태 관리"
- "전역 상태 관리", "Redux 대신 뭐 써", "Zustand store 만들어줘"
- "전역 상태 어떻게 써", "Zustand persist", "Zustand devtools"

---

## 클라이언트 상태 vs 서버 상태 구분

| 상태 유형 | 예시 | 권장 관리 도구 |
|:---|:---|:---|
| **서버 상태** | API에서 가져온 사용자 목록, 상세 정보 | TanStack Query (React Query) |
| **클라이언트 UI 상태** | 사이드바 열림/닫힘, 현재 선택된 탭, 모달 표시 여부 | Zustand 또는 useState |
| **인증 상태** | 로그인 사용자 정보, 권한 | Zustand (persist 미들웨어) |
| **전역 앱 설정** | 테마(다크/라이트), 언어, 알림 여부 | Zustand (persist 미들웨어) |
| **폼 상태** | 입력값, 유효성 에러 | React Hook Form |

> **핵심 원칙**: API 데이터는 Zustand에 넣지 말고 React Query로 관리. Zustand는 순수 클라이언트 상태만 담는다.

---

## Zustand 기본 store 정의 (`src/store/authStore.ts`)

```typescript
import { create } from 'zustand'
import { devtools, persist } from 'zustand/middleware'
import { immer } from 'zustand/middleware/immer'

interface AuthUser {
  userId: number
  userNm: string
  email: string
  roles: string[]
}

interface AuthState {
  user: AuthUser | null
  isAuthenticated: boolean
  setUser: (user: AuthUser) => void
  clearUser: () => void
}

export const useAuthStore = create<AuthState>()(
  devtools(
    persist(
      immer((set) => ({
        user: null,
        isAuthenticated: false,

        setUser: (user) =>
          set((state) => {
            state.user = user
            state.isAuthenticated = true
          }),

        clearUser: () =>
          set((state) => {
            state.user = null
            state.isAuthenticated = false
          }),
      })),
      {
        name: 'auth-storage',          // localStorage 키
        partialize: (state) => ({       // 저장할 필드만 선택
          user: state.user,
          isAuthenticated: state.isAuthenticated,
        }),
      }
    ),
    { name: 'AuthStore' }  // DevTools 표시명
  )
)
```

---

## UI 상태 store (`src/store/uiStore.ts`)

```typescript
import { create } from 'zustand'
import { devtools } from 'zustand/middleware'

interface UIState {
  sidebarOpen: boolean
  activeTab: string
  toggleSidebar: () => void
  setActiveTab: (tab: string) => void
}

export const useUIStore = create<UIState>()(
  devtools(
    (set) => ({
      sidebarOpen: true,
      activeTab: 'dashboard',
      toggleSidebar: () => set((state) => ({ sidebarOpen: !state.sidebarOpen })),
      setActiveTab: (tab) => set({ activeTab: tab }),
    }),
    { name: 'UIStore' }
  )
)
```

---

## 컴포넌트에서 사용

```tsx
// 전체 상태 구독 (권장 X — 불필요한 리렌더 발생)
const store = useAuthStore()

// 특정 값만 선택적 구독 (권장 — 해당 값 변경 시만 리렌더)
const user = useAuthStore((state) => state.user)
const isAuthenticated = useAuthStore((state) => state.isAuthenticated)
const setUser = useAuthStore((state) => state.setUser)

// 여러 값 동시 선택 (shallow 비교 사용)
import { useShallow } from 'zustand/react/shallow'

const { user, isAuthenticated } = useAuthStore(
  useShallow((state) => ({ user: state.user, isAuthenticated: state.isAuthenticated }))
)
```

---

## 미들웨어 조합 순서

```typescript
// 권장 조합 순서: devtools → persist → immer
create<State>()(
  devtools(
    persist(
      immer(
        (set, get) => ({ /* store */ })
      ),
      { name: 'storage-key' }
    ),
    { name: 'StoreName' }
  )
)

// 설치
// npm install zustand
// npm install immer  (immer 미들웨어 사용 시)
```

---

## slice 패턴으로 도메인 분리

```typescript
// src/store/slices/notificationSlice.ts
import type { StateCreator } from 'zustand'

interface Notification {
  id: string
  message: string
  type: 'success' | 'error' | 'info'
}

export interface NotificationSlice {
  notifications: Notification[]
  addNotification: (n: Omit<Notification, 'id'>) => void
  removeNotification: (id: string) => void
}

export const createNotificationSlice: StateCreator<NotificationSlice> = (set) => ({
  notifications: [],
  addNotification: (n) =>
    set((state) => ({
      notifications: [...state.notifications, { ...n, id: Date.now().toString() }],
    })),
  removeNotification: (id) =>
    set((state) => ({ notifications: state.notifications.filter((n) => n.id !== id) })),
})

// src/store/rootStore.ts — 슬라이스 합치기
import { create } from 'zustand'
import { createNotificationSlice, type NotificationSlice } from './slices/notificationSlice'

type RootStore = NotificationSlice /* & OtherSlice */

export const useRootStore = create<RootStore>()((...a) => ({
  ...createNotificationSlice(...a),
}))
```

---

## Redux Toolkit 선택 기준

Zustand 대신 Redux Toolkit을 선택해야 하는 경우:

| 상황 | 권장 |
|:---|:---|
| 팀이 Redux에 이미 익숙하고 기존 Redux 코드베이스 | Redux Toolkit |
| 복잡한 미들웨어(redux-saga, redux-observable) 필요 | Redux Toolkit |
| RTK Query를 React Query 대신 사용 | Redux Toolkit |
| 단순 전역 상태(인증, UI 토글) | **Zustand** |
| 신규 프로젝트, 소규모 팀 | **Zustand** |

---

## 상태 초기화 패턴

```typescript
// 로그아웃 시 스토어 전체 초기화
const clearAllStores = () => {
  useAuthStore.getState().clearUser()
  useUIStore.setState({ sidebarOpen: true, activeTab: 'dashboard' })
  // React Query 캐시 초기화
  queryClient.clear()
}
```
