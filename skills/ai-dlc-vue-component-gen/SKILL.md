---
name: ai-dlc-vue-component-gen
description: AI-DLC 개발단계(프론트엔드-Vue.js) 스킬. SFC 컴포넌트·Composable·Pinia 스토어 코드를 생성한다. "Vue 컴포넌트 만들어줘", "SFC 생성", "Composable 만들어줘", "Vue 화면 구현", "뷰 컴포넌트 코드 작성", "Vue 화면 코드 생성", "Vue 페이지 구현" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Vue.js SFC 컴포넌트 생성

화면설계서(SCR-NNN)·API설계서를 기반으로 Vue 3 SFC 컴포넌트, Composable, Pinia 스토어 코드를 생성한다.
공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "Vue 컴포넌트 만들어줘", "SFC 생성", "Composable 만들어줘"
- "Vue 화면 구현", "뷰 컴포넌트 코드 작성", "Vue 화면 코드 생성"
- "Vue 목록 화면 만들어줘", "Vue 폼 화면 구현"

---

## 입력

### 필수
- 화면설계서(SCR-NNN) 또는 화면 요구사항 (화면 목적, 주요 기능)
- API 설계서 또는 API 명세 (엔드포인트, 요청·응답 스키마)
- 프로젝트 구조 (`ai-dlc-vue-project-setup` 산출물 참조)

### 선택
- 기존 프로젝트 코드 (일관성 유지를 위해 읽기)
- Vue 구현계획서 (`ai-dlc-vue-impl-plan` 산출물)

---

## 분석 절차

### 1단계: 화면 목적 파악
- 화면 유형 결정: LIST / FORM / DETAIL / DASHBOARD / AUTH
- 필요한 API 엔드포인트 목록 정리
- 필요한 컴포넌트 계층 결정 (View / Container / Presentational)

### 2단계: 컴포넌트 구조 설계
각 컴포넌트 역할 분리:
- **View** (`src/views/XxxView.vue`): 라우트 진입점, Composable 호출, 레이아웃
- **Container** (`src/components/XxxContainer.vue`): 데이터 흐름 관리, 이벤트 처리 (필요 시)
- **Presentational** (`src/components/XxxTable.vue` 등): 순수 UI, props/emits만 사용

### 3단계: SFC 코드 작성 원칙

#### `<script setup lang="ts">` 필수
```vue
<script setup lang="ts">
// 1. 외부 라이브러리 import
// 2. 내부 모듈 import (@/ alias 사용)
// 3. props/emits 정의
// 4. Composable/Store 사용
// 5. 로컬 상태 (ref/reactive)
// 6. computed/watch
// 7. 함수 정의
// 8. lifecycle hooks (onMounted 등)
</script>

<template>
  <!-- 최상위 단일 엘리먼트 또는 Fragment(여러 엘리먼트) -->
</template>

<style scoped>
/* 꼭 필요한 스타일만; 가능하면 Tailwind 사용 */
</style>
```

#### Props/Emits 타입 정의
```typescript
// defineProps — 타입 제네릭 방식 사용
const props = defineProps<{
  userId: number
  readonly?: boolean
}>()

// withDefaults로 기본값 설정
const props = withDefaults(defineProps<{
  items: User[]
  loading?: boolean
}>(), {
  loading: false,
})

// defineEmits — 타입 제네릭 방식 사용
const emit = defineEmits<{
  (e: 'update', id: number, data: UpdateUserDto): void
  (e: 'delete', id: number): void
}>()
```

#### Composable 패턴
```typescript
// src/composables/useUserList.ts
import { ref, computed } from 'vue'
import { useQuery } from '@tanstack/vue-query'
import { fetchUserList } from '@/api/user.api'

export function useUserList() {
  const filters = ref<UserFilters>({ page: 1, size: 20 })

  const { data, isLoading, isError, error } = useQuery({
    queryKey: ['users', 'list', filters],
    queryFn: () => fetchUserList(filters.value),
  })

  const users = computed(() => data.value?.content ?? [])
  const totalCount = computed(() => data.value?.totalElements ?? 0)

  return { users, totalCount, isLoading, isError, error, filters }
}
```

#### Pinia 스토어 연동 (storeToRefs 필수)
```typescript
import { storeToRefs } from 'pinia'
import { useAuthStore } from '@/stores/auth'

const authStore = useAuthStore()
const { user, isAuthenticated } = storeToRefs(authStore) // 반응성 유지
const { login, logout } = authStore // actions는 직접 구조분해 가능
```

### 4단계: Vue Query 데이터 패칭

```typescript
// 목록 조회
const { data, isLoading, isError } = useQuery({
  queryKey: ['users', 'list', filters],
  queryFn: () => fetchUserList(filters.value),
})

// 단건 조회
const { data: user } = useQuery({
  queryKey: ['users', 'detail', userId],
  queryFn: () => fetchUserById(userId.value),
  enabled: computed(() => !!userId.value),
})

// mutation (생성/수정/삭제)
const queryClient = useQueryClient()
const { mutate: createUser, isPending } = useMutation({
  mutationFn: createUserApi,
  onSuccess: () => {
    queryClient.invalidateQueries({ queryKey: ['users', 'list'] })
    router.push('/users')
  },
})
```

### 5단계: VeeValidate + Zod 폼 처리

```typescript
import { useForm } from 'vee-validate'
import { toTypedSchema } from '@vee-validate/zod'
import { z } from 'zod'

const schema = z.object({
  name: z.string().min(1, '이름을 입력하세요').max(50),
  email: z.string().email('올바른 이메일 형식이 아닙니다'),
})

const { handleSubmit, errors, isSubmitting } = useForm({
  validationSchema: toTypedSchema(schema),
})

const onSubmit = handleSubmit(async (values) => {
  await createUser(values)
})
```

### 6단계: v-for :key 원칙
- `v-for`는 반드시 `:key` 사용
- key는 고유 식별자(id) 사용; index 사용 금지 (순서 변경 시 버그 유발)

---

## 생성 원칙

| 원칙 | 내용 |
|:---|:---|
| `<script setup>` 필수 | Options API 사용 금지 |
| 컴포넌트 최대 150줄 | 초과 시 Composable/하위 컴포넌트로 분리 |
| Composable 추출 | 10줄 이상 로직은 `useXxx.ts`로 분리 |
| storeToRefs | Pinia state/getters는 반드시 storeToRefs로 구조분해 |
| v-for + :key | index 대신 data.id 사용 |
| TypeScript strict | any 사용 금지, 모든 props/emits 타입 명시 |
| Tailwind 클래스 | 인라인 style 금지, cn() 함수 활용 |

---

## 산출물

- `src/views/XxxView.vue` — View 컴포넌트 (라우트 단위)
- `src/components/XxxTable.vue` / `XxxForm.vue` — Presentational 컴포넌트
- `src/composables/useXxx.ts` — 재사용 로직 Composable
- `src/stores/xxxStore.ts` — Pinia UI 상태 스토어 (필요 시)
- `src/api/xxx.api.ts` — API 함수 모듈 (필요 시)
- `src/types/xxx.types.ts` — 타입 정의 (필요 시)

template.md에서 각 파일의 기본 코드 패턴을 참조한다.
