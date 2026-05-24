---
name: ai-dlc-vue-form-guide
description: AI-DLC 개발단계(프론트엔드-Vue.js) 가이드 스킬. VeeValidate v4 + Zod 폼 검증 사용법을 안내한다. "VeeValidate 사용법", "Vue 폼 검증", "Zod Vue 폼", "useForm VeeValidate", "toTypedSchema", "Vue 폼 유효성 검사" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC VeeValidate v4 + Zod 폼 검증 가이드

Vue.js에서 VeeValidate v4와 Zod 스키마를 사용하여 타입 안전한 폼 검증을 구현하는 방법을 안내한다.

## 트리거

- "VeeValidate 사용법", "Vue 폼 검증", "Zod Vue 폼"
- "useForm VeeValidate", "toTypedSchema", "Vue 폼 유효성 검사"
- "Vue 폼 에러 메시지", "VeeValidate Zod 통합"

---

## 설치

```bash
npm install vee-validate @vee-validate/zod zod
```

---

## 기본 패턴 (useForm + useField)

```vue
<script setup lang="ts">
import { useForm, useField } from 'vee-validate'
import { toTypedSchema } from '@vee-validate/zod'
import { z } from 'zod'

// 1. Zod 스키마 정의
const schema = z.object({
  name: z.string().min(1, '이름을 입력하세요').max(50, '이름은 50자 이내'),
  email: z.string().email('올바른 이메일 형식이 아닙니다'),
  age: z.number({ required_error: '나이를 입력하세요' })
    .min(1, '나이는 1 이상이어야 합니다')
    .max(150, '올바른 나이를 입력하세요'),
  role: z.enum(['ADMIN', 'USER'], { required_error: '역할을 선택하세요' }),
})

type FormValues = z.infer<typeof schema>

// 2. useForm 설정
const { handleSubmit, errors, resetForm, isSubmitting, setValues } = useForm<FormValues>({
  validationSchema: toTypedSchema(schema),
  initialValues: {
    name: '',
    email: '',
    role: 'USER',
  },
})

// 3. useField로 개별 필드 바인딩
const { value: name, errorMessage: nameError } = useField<string>('name')
const { value: email, errorMessage: emailError } = useField<string>('email')
const { value: age, errorMessage: ageError } = useField<number>('age')
const { value: role, errorMessage: roleError } = useField<string>('role')

// 4. submit 핸들러 (유효성 통과 시에만 실행)
const onSubmit = handleSubmit(async (values: FormValues) => {
  await createUser(values)
})

// 수정 시 기존 값 주입
function loadUserData(user: User) {
  setValues({
    name: user.name,
    email: user.email,
    role: user.role,
  })
}
</script>

<template>
  <form @submit="onSubmit" novalidate>
    <div>
      <label for="name">이름 <span>*</span></label>
      <input id="name" v-model="name" type="text" />
      <span v-if="nameError" role="alert">{{ nameError }}</span>
    </div>

    <div>
      <label for="email">이메일 <span>*</span></label>
      <input id="email" v-model="email" type="email" />
      <span v-if="emailError" role="alert">{{ emailError }}</span>
    </div>

    <button type="submit" :disabled="isSubmitting">
      {{ isSubmitting ? '저장 중...' : '저장' }}
    </button>
  </form>
</template>
```

---

## Field 컴포넌트 방식 (대안)

```vue
<template>
  <form @submit="onSubmit">
    <Field name="email" v-slot="{ field, errorMessage }">
      <input v-bind="field" type="email" />
      <span v-if="errorMessage" role="alert">{{ errorMessage }}</span>
    </Field>
    <ErrorMessage name="email" />
  </form>
</template>

<script setup lang="ts">
import { Field, ErrorMessage, useForm } from 'vee-validate'
import { toTypedSchema } from '@vee-validate/zod'
</script>
```

---

## 비동기 검증 (서버 중복 확인)

```typescript
const schema = z.object({
  email: z
    .string()
    .email('올바른 이메일 형식이 아닙니다')
    .refine(
      async (email) => {
        // 서버에서 중복 확인
        const { isDuplicate } = await checkEmailDuplicate(email)
        return !isDuplicate
      },
      { message: '이미 사용 중인 이메일입니다' },
    ),
})
```

---

## shadcn-vue Input 연동

```vue
<script setup lang="ts">
import { useField } from 'vee-validate'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import { cn } from '@/utils/cn'

const { value, errorMessage, meta } = useField<string>('email')
</script>

<template>
  <div class="space-y-2">
    <Label for="email">이메일 <span class="text-destructive">*</span></Label>
    <Input
      id="email"
      v-model="value"
      type="email"
      :class="cn(errorMessage && 'border-destructive focus-visible:ring-destructive')"
    />
    <p v-if="errorMessage" class="text-sm text-destructive" role="alert">
      {{ errorMessage }}
    </p>
  </div>
</template>
```

---

## Select 컴포넌트 연동 (shadcn-vue)

```vue
<script setup lang="ts">
import { useField } from 'vee-validate'
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select'

const { value: role, errorMessage: roleError } = useField<string>('role')
</script>

<template>
  <div class="space-y-2">
    <Label>역할</Label>
    <Select v-model="role">
      <SelectTrigger :class="cn(roleError && 'border-destructive')">
        <SelectValue placeholder="역할을 선택하세요" />
      </SelectTrigger>
      <SelectContent>
        <SelectItem value="USER">일반 사용자</SelectItem>
        <SelectItem value="ADMIN">관리자</SelectItem>
      </SelectContent>
    </Select>
    <p v-if="roleError" class="text-sm text-destructive" role="alert">{{ roleError }}</p>
  </div>
</template>
```

---

## 폼 초기화 및 외부 오류 처리

```typescript
const { handleSubmit, resetForm, setErrors, setFieldError } = useForm(...)

// 폼 초기화
function cancel() {
  resetForm()
}

// 서버 오류를 필드에 반영
try {
  await onSubmit(values)
} catch (err) {
  if (err.response?.data?.fieldErrors) {
    setErrors(err.response.data.fieldErrors)
    // 예: { email: '이미 사용 중인 이메일', name: '...' }
  }
}

// 특정 필드 오류만 설정
setFieldError('email', '이미 사용 중인 이메일입니다')
```

---

## 자주 쓰는 Zod 패턴

```typescript
const schema = z.object({
  // 선택 필드
  description: z.string().optional(),

  // null 허용
  managerId: z.number().nullable(),

  // 숫자 문자열 변환
  price: z.coerce.number().min(0, '가격은 0 이상이어야 합니다'),

  // 날짜 문자열
  birthDate: z.string().regex(/^\d{4}-\d{2}-\d{2}$/, 'YYYY-MM-DD 형식으로 입력하세요'),

  // 조건부 검증
  confirmPassword: z.string(),
}).refine(
  (data) => data.password === data.confirmPassword,
  { message: '비밀번호가 일치하지 않습니다', path: ['confirmPassword'] },
)
```
