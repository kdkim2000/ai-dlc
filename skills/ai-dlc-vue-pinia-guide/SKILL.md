---
name: ai-dlc-vue-pinia-guide
description: AI-DLC 개발단계(프론트엔드-Vue.js) 가이드 스킬. Pinia 상태관리 사용법을 안내한다. "Pinia 사용법", "Vue 상태관리 방법", "Pinia 스토어 만들기", "storeToRefs 사용법", "Pinia 영속성", "뷰 상태관리 가이드", "Pinia defineStore" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Pinia 상태관리 가이드

Vue.js 공식 상태관리 라이브러리 Pinia의 설정·패턴·영속성을 안내한다.

## 트리거

- "Pinia 사용법", "Vue 상태관리 방법", "Pinia 스토어 만들기"
- "storeToRefs 사용법", "Pinia 영속성", "Pinia defineStore"
- "뷰 상태관리 가이드", "Pinia 스토어 예시"

---

## Pinia vs Vuex

| 항목 | Pinia | Vuex 4 |
|:---|:---|:---|
| Vue 3 공식 권장 | O | X (레거시) |
| TypeScript | 완전 지원 | 제한적 |
| Devtools | O | O |
| 모듈 구조 | 스토어 파일 분리 | modules 중첩 |
| Mutations | 없음 (actions로 통합) | 있음 |
| Composition API | Options / Composition 둘 다 | 제한적 |

---

## defineStore 패턴

### Composition API 방식 (권장)

```typescript
// src/stores/auth.ts
import { defineStore } from 'pinia'
import { ref, computed } from 'vue'
import type { User } from '@/types/auth'

export const useAuthStore = defineStore('auth', () => {
  // State: ref() 또는 reactive() 사용
  const token = ref<string | null>(null)
  const user = ref<User | null>(null)

  // Getters: computed()
  const isAuthenticated = computed(() => !!token.value)
  const displayName = computed(() => user.value?.name ?? '게스트')

  // Actions: 일반 함수
  function setToken(newToken: string) {
    token.value = newToken
  }

  async function login(email: string, password: string) {
    // API 호출 등 비동기 액션
    const response = await loginApi(email, password)
    token.value = response.token
    user.value = response.user
  }

  function logout() {
    token.value = null
    user.value = null
  }

  return { token, user, isAuthenticated, displayName, setToken, login, logout }
})
```

### Options API 방식 (참고)

```typescript
export const useCounterStore = defineStore('counter', {
  state: () => ({ count: 0 }),
  getters: {
    doubled: (state) => state.count * 2,
  },
  actions: {
    increment() { this.count++ },
  },
})
```

---

## 컴포넌트에서 사용

### storeToRefs 필수 사용

```vue
<script setup lang="ts">
import { storeToRefs } from 'pinia'
import { useAuthStore } from '@/stores/auth'

const authStore = useAuthStore()

// ✅ 올바른 방법: storeToRefs로 반응성 유지
const { token, user, isAuthenticated } = storeToRefs(authStore)

// ✅ Actions는 직접 구조분해 (반응성 불필요)
const { login, logout } = authStore

// ❌ 잘못된 방법: 반응성 깨짐
// const { token } = authStore  // token이 reactive하지 않게 됨
</script>
```

---

## 스토어 간 참조

```typescript
// src/stores/ui.ts
import { defineStore } from 'pinia'
import { useAuthStore } from './auth'  // 다른 스토어 참조 가능

export const useUiStore = defineStore('ui', () => {
  const authStore = useAuthStore()  // 스토어 내부에서 사용

  const sidebarOpen = ref(false)
  const welcomeMessage = computed(
    () => `안녕하세요, ${authStore.displayName}님`
  )

  return { sidebarOpen, welcomeMessage }
})
```

---

## 영속성 (pinia-plugin-persistedstate)

```typescript
// src/main.ts
import { createPinia } from 'pinia'
import piniaPluginPersistedstate from 'pinia-plugin-persistedstate'

const pinia = createPinia()
pinia.use(piniaPluginPersistedstate)

// 스토어에서 영속성 설정
export const useAuthStore = defineStore(
  'auth',
  () => {
    const token = ref<string | null>(null)
    const user = ref<User | null>(null)
    // ...
    return { token, user }
  },
  {
    persist: {
      key: 'auth',           // localStorage key (기본값: store id)
      storage: localStorage,  // sessionStorage도 가능
      paths: ['token', 'user'], // 영속할 state 목록 (생략 시 전체)
    },
  },
)
```

---

## Pinia 설계 원칙

| 원칙 | 설명 |
|:---|:---|
| 서버 데이터는 Vue Query | API 응답 데이터는 Pinia가 아닌 `useQuery`로 관리 |
| UI 상태만 Pinia | 사이드바 열림·닫힘, 테마, 선택된 항목 ID 등 UI 상태 |
| 인증 정보는 Pinia | 토큰·사용자 정보 (전역 상태 + 영속성 필요) |
| 스토어 분리 | 도메인별 파일 분리 (`auth.ts`, `ui.ts`, `notification.ts`) |
| storeToRefs 필수 | state/getters를 구조분해할 때 반드시 storeToRefs 사용 |

---

## Pinia DevTools

Vue DevTools 확장에서 Pinia 탭:
- 스토어 상태 실시간 확인·수정
- 액션 타임라인 추적
- 스냅샷 저장

```typescript
// Hot Module Replacement (HMR) 지원 — 개발 중 스토어 수정 시 상태 유지
if (import.meta.hot) {
  import.meta.hot.accept(acceptHMRUpdate(useAuthStore, import.meta.hot))
}
```
