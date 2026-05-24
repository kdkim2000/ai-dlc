---
name: ai-dlc-vue-code-revise
description: AI-DLC 개발단계(프론트엔드-Vue.js) 스킬. Vue.js 코드 리뷰 결과(VV-001~010)를 소스코드에 반영한다. "Vue 코드 수정해줘", "Vue 리뷰 결과 반영", "VV 이슈 코드 수정", "Vue 코드 개선해줘", "뷰 코드 리팩터링", "Vue 코드 품질 개선" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Vue.js 코드 수정 (리뷰 결과 반영)

`ai-dlc-vue-code-review` / `ai-dlc-vue-ts-check` / `ai-dlc-vue-lint-check` 결과를 소스코드에 반영한다.
공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "Vue 코드 수정해줘", "Vue 리뷰 결과 반영", "VV 이슈 코드 수정"
- "Vue 코드 개선해줘", "뷰 코드 리팩터링", "Vue 코드 품질 개선"
- "VV-NNN 수정해줘", "Vue 이슈 수정"

---

## 입력

### 필수
- 수정 대상 소스 파일 경로
- 검토 결과 보고서 (`ai-dlc-vue-code-review` 산출물) 또는 이슈 코드 목록

### 선택
- 수정 범위 (특정 파일, 특정 이슈 코드)

---

## 수정 우선순위

아래 순서로 수정:

1. **아키텍처·런타임 오류** (VV-003/VV-008/VV-009/VV-010, SC-001/SC-002)
   - 데이터 흐름 구조 문제 → Pinia·Composable·Vue Query 도입
   - `v-for` :key 오류 → id로 교체
   - API 직접 호출 → Composable + useQuery로 분리
   - 폼 검증 누락 → VeeValidate + Zod 도입
   - 보안 이슈 → 즉시 수정

2. **코드 스타일** (VV-001/VV-002)
   - Options API → Composition API (`<script setup>`) 전환
   - `setup()` 반환 패턴 → `<script setup>` 리팩터링

3. **타입 안전성** (TC-001/TC-002/TC-003, VV-005)
   - `any` 타입 → 구체적 타입으로 교체
   - `defineProps`/`defineEmits` 타입 추가

4. **Composable 추출** (VV-004)
   - 컴포넌트 내 복잡 로직 → `useXxx.ts`로 분리

5. **기타** (VV-006/VV-007, PF-001~003, A11Y)
   - `watchEffect` → `watch`로 교체
   - `$router` → `useRouter()` 교체
   - 성능 최적화
   - 접근성 개선

---

## 수정 패턴

### VV-008: v-for :key 수정
```vue
<!-- 수정 전 -->
<li v-for="(item, index) in items" :key="index">

<!-- 수정 후 -->
<li v-for="item in items" :key="item.id">
```

### VV-009: 컴포넌트 내 API 직접 호출 → Composable 분리
```vue
<!-- 수정 전: View 내 직접 호출 -->
<script setup lang="ts">
import axios from 'axios'
const users = ref([])
onMounted(async () => {
  const { data } = await axios.get('/api/users')
  users.value = data
})
</script>

<!-- 수정 후: Composable + Vue Query -->
<script setup lang="ts">
import { useUserList } from '@/composables/useUserList'
const { users, isLoading, isError } = useUserList()
</script>
```

### VV-010: 폼 검증 누락 → VeeValidate + Zod 도입
```vue
<!-- 수정 전 -->
<script setup lang="ts">
async function onSubmit() {
  await createUser(formData.value) // 검증 없음
}
</script>

<!-- 수정 후 -->
<script setup lang="ts">
import { useForm } from 'vee-validate'
import { toTypedSchema } from '@vee-validate/zod'
const { handleSubmit, errors } = useForm({
  validationSchema: toTypedSchema(schema),
})
const onSubmit = handleSubmit(async (values) => {
  await createUser(values)
})
</script>
```

### VV-001/VV-002: Options API → `<script setup>` 전환
```vue
<!-- 수정 전 (Options API) -->
<script lang="ts">
export default {
  data() { return { count: 0 } },
  methods: { increment() { this.count++ } },
}
</script>

<!-- 수정 후 (<script setup>) -->
<script setup lang="ts">
import { ref } from 'vue'
const count = ref(0)
function increment() { count.value++ }
</script>
```

### VV-005: defineProps 타입 추가
```typescript
// 수정 전
const props = defineProps(['userId', 'readonly'])

// 수정 후
const props = defineProps<{
  userId: number
  readonly?: boolean
}>()
```

### TC-001: any 타입 제거
```typescript
// 수정 전
const data = ref<any>(null)

// 수정 후
const data = ref<User | null>(null)
```

---

## 수정 절차

### 1단계: 이슈 목록 확인
검토 보고서의 이슈를 우선순위 순으로 정리.

### 2단계: 파일별 수정 실행
- 아키텍처 변경이 필요한 경우 (VV-009, VV-003): 새 Composable/Store 파일 생성 후 기존 파일 수정
- 패턴 교체인 경우 (VV-001, VV-005, VV-008): 파일 직접 Edit

### 3단계: 수정 결과 요약
수정한 파일 목록, 이슈 코드별 처리 결과 정리.

---

## 산출물

- 수정된 소스 파일 (`.vue`, `.ts`)
- 수정 요약 (어떤 이슈를 어떤 방법으로 수정했는지)
