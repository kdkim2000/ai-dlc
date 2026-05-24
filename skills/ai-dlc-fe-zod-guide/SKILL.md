---
name: ai-dlc-fe-zod-guide
description: AI-DLC 개발단계(프론트엔드-React) 스킬. Zod 스키마 검증 활용 가이드를 제공한다. "Zod 가이드", "스키마 검증 방법", "폼 유효성 검사", "입력값 검증 방법", "Zod 스키마 만들어줘", "zodResolver 사용법", "Zod 에러 메시지 한국어" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Zod 스키마 검증 활용 가이드

Zod 스키마 정의·React Hook Form 연동·에러 메시지 한국어 설정·서버 응답 검증 패턴을 대화창에 출력한다. 파일을 수정하지 않는다.

## 트리거

- "Zod 가이드", "스키마 검증 방법", "폼 유효성 검사"
- "입력값 검증 방법", "Zod 스키마 만들어줘", "zodResolver 사용법"
- "Zod 에러 메시지 한국어", "z.infer 타입 추출", "Zod enum"

---

## 기본 스키마 정의

```typescript
import { z } from 'zod'

// 기본 타입
const schema = z.object({
  userNm: z.string().min(1, '사용자명을 입력하세요.').max(50, '50자 이내로 입력하세요.'),
  email: z.string().min(1, '이메일을 입력하세요.').email('이메일 형식이 올바르지 않습니다.'),
  age: z.number().int().min(1, '1 이상의 숫자를 입력하세요.').max(150),
  phone: z.string().regex(/^010-\d{4}-\d{4}$/, '010-XXXX-XXXX 형식으로 입력하세요.').optional(),
})

// TypeScript 타입 추출
type UserForm = z.infer<typeof schema>
// → { userNm: string; email: string; age: number; phone?: string }
```

---

## 파일 스키마 구성 (`src/types/user.schema.ts`)

```typescript
import { z } from 'zod'

// 등록 스키마
export const userCreateSchema = z.object({
  userNm: z.string().min(1, '사용자명을 입력하세요.').max(50),
  email: z.string().min(1, '이메일을 입력하세요.').email('이메일 형식이 올바르지 않습니다.'),
  deptCd: z.string().min(1, '부서를 선택하세요.'),
  useYn: z.enum(['Y', 'N']).default('Y'),
})

// 수정 스키마 — 일부 필드 선택적
export const userUpdateSchema = userCreateSchema
  .omit({ email: true })  // 이메일 수정 불가
  .extend({ userId: z.number() })

// 검색 스키마
export const userFilterSchema = z.object({
  userNm: z.string().optional(),
  useYn: z.enum(['Y', 'N', '']).optional(),
  page: z.number().default(0),
  size: z.number().default(20),
})

// 타입 export
export type UserCreateReq = z.infer<typeof userCreateSchema>
export type UserUpdateReq = z.infer<typeof userUpdateSchema>
export type UserFilter = z.infer<typeof userFilterSchema>
```

---

## React Hook Form + zodResolver 연동

```tsx
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { userCreateSchema, type UserCreateReq } from '@/types/user.schema'

function UserCreateForm() {
  const form = useForm<UserCreateReq>({
    resolver: zodResolver(userCreateSchema),
    defaultValues: {
      userNm: '',
      email: '',
      deptCd: '',
      useYn: 'Y',
    },
  })

  const onSubmit = (data: UserCreateReq) => {
    // data는 이미 Zod로 검증된 타입
    createUser(data)
  }

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <input {...form.register('userNm')} />
      {form.formState.errors.userNm && (
        <p>{form.formState.errors.userNm.message}</p>
      )}
      {/* shadcn/ui FormMessage가 에러를 자동 표시 */}
    </form>
  )
}
```

---

## 고급 유효성 패턴

### 비밀번호 확인 (superRefine)
```typescript
const passwordSchema = z
  .object({
    password: z.string().min(8, '8자 이상 입력하세요.'),
    passwordConfirm: z.string(),
  })
  .superRefine(({ password, passwordConfirm }, ctx) => {
    if (password !== passwordConfirm) {
      ctx.addIssue({
        code: z.ZodIssueCode.custom,
        message: '비밀번호가 일치하지 않습니다.',
        path: ['passwordConfirm'],
      })
    }
  })
```

### 조건부 필드
```typescript
const schema = z.object({
  useYn: z.enum(['Y', 'N']),
  endDt: z.string().optional(),
}).refine(
  (data) => data.useYn !== 'N' || !!data.endDt,
  { message: '비활성화 시 종료일을 입력하세요.', path: ['endDt'] }
)
```

### 숫자 문자열 변환
```typescript
// HTML input은 항상 string → coerce로 변환
const schema = z.object({
  count: z.coerce.number().int().min(0),
  price: z.coerce.number().positive('양수를 입력하세요.'),
})
```

---

## 서버 응답 타입 검증

```typescript
import { z } from 'zod'

const userVOSchema = z.object({
  userId: z.number(),
  userNm: z.string(),
  email: z.string().email(),
  useYn: z.enum(['Y', 'N']),
  createdAt: z.string(),
})

// API 응답 파싱 (런타임 타입 검증)
const parsed = userVOSchema.safeParse(response.data)
if (!parsed.success) {
  console.error('API 응답 스키마 불일치:', parsed.error.issues)
} else {
  const user = parsed.data  // 타입: UserVO
}
```

---

## 에러 메시지 전역 한국어 설정

```typescript
// src/main.tsx 또는 src/lib/zod.ts
import { z } from 'zod'
import { zodI18nMap } from 'zod-i18n-map'
import translation from 'zod-i18n-map/locales/ko/zod.json'
import i18next from 'i18next'

i18next.init({ lng: 'ko', resources: { ko: { zod: translation } } })
z.setErrorMap(zodI18nMap)

// 의존성 추가 필요:
// npm install zod-i18n-map i18next
```

---

## Enum 타입 정의

```typescript
// Zod enum → TypeScript 타입 동시 생성
export const UseYn = z.enum(['Y', 'N'])
export type UseYnType = z.infer<typeof UseYn>  // 'Y' | 'N'

export const DeptCd = z.enum(['DEV', 'HR', 'SALES', 'IT'])
export type DeptCdType = z.infer<typeof DeptCd>
```
