---
name: ai-dlc-nxt-server-action-gen
description: AI-DLC 개발단계(프론트엔드-Next.js) 스킬. Next.js Server Actions('use server') 코드를 생성한다. "Server Action 만들어줘", "서버 액션 생성", "폼 서버 처리 코드", "use server 코드", "Server Action 구현" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Next.js Server Actions 코드 생성

유즈케이스(UC-NNN)·API 설계서를 입력받아 `actions/` 폴더의 Server Actions 코드를 생성한다.

## 트리거

- "Server Action 만들어줘", "서버 액션 생성"
- "폼 서버 처리 코드", "use server 코드"
- "Server Action 구현", "뮤테이션 서버 처리"

---

## 입력

### 필수
- 도메인명 (예: user, product, order)
- 수행할 작업 목록 (생성/수정/삭제)

### 선택
- 유즈케이스 문서 (UC-NNN)
- Zod 스키마 (없으면 새로 작성)
- Prisma 모델명 (없으면 fetch API 호출로 대체)

---

## 생성 절차

1. **`actions/[domain].ts` 파일 생성**: 파일 상단에 `'use server'` 지시어
2. **FormState 타입 정의**: `{ error: string | null; success?: boolean }`
3. **Zod 스키마 작성**: 입력 유효성 검사 스키마
4. **각 Action 함수 작성**:
   - Zod `safeParse` → 실패 시 에러 반환
   - DB 처리 (Prisma) 또는 외부 API 호출
   - `revalidatePath` / `revalidateTag` 캐시 무효화
   - 성공 시 `redirect()` 또는 FormState 반환
5. **CC 연동 예시 작성**: `useFormState` + `useFormStatus` 사용법

---

## Server Action 기본 패턴

```typescript
'use server'

import { z } from 'zod'
import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'
import { auth } from '@/lib/auth'

const CreateUserSchema = z.object({
  name: z.string().min(2, '이름은 2자 이상이어야 합니다'),
  email: z.string().email('유효한 이메일을 입력하세요'),
  role: z.enum(['ADMIN', 'USER']),
})

export type FormState = {
  error: string | null
  fieldErrors?: Record<string, string[]>
}

export async function createUser(
  _prevState: FormState,
  formData: FormData,
): Promise<FormState> {
  const session = await auth()
  if (!session) return { error: '인증이 필요합니다' }

  const parsed = CreateUserSchema.safeParse({
    name: formData.get('name'),
    email: formData.get('email'),
    role: formData.get('role'),
  })

  if (!parsed.success) {
    return {
      error: '입력값을 확인해 주세요',
      fieldErrors: parsed.error.flatten().fieldErrors,
    }
  }

  try {
    await prisma.user.create({ data: parsed.data })
  } catch {
    return { error: '사용자 등록에 실패했습니다' }
  }

  revalidatePath('/users')
  redirect('/users')
}
```

---

## 캐시 무효화 전략

| 상황 | 사용 함수 | 예시 |
|:---|:---|:---|
| 특정 경로 목록 갱신 | `revalidatePath` | `revalidatePath('/users')` |
| 상세 페이지 캐시 갱신 | `revalidatePath` | `revalidatePath('/users/[id]', 'page')` |
| 태그 기반 캐시 무효화 | `revalidateTag` | `revalidateTag('users')` |
| 생성 후 목록으로 이동 | `redirect` | `redirect('/users')` |

---

## CC 연동 패턴

```typescript
'use client'

import { useFormState, useFormStatus } from 'react-dom'
import { createUser, type FormState } from '@/actions/user'

const initialState: FormState = { error: null }

export function CreateUserForm() {
  const [state, action] = useFormState(createUser, initialState)

  return (
    <form action={action} className="space-y-4">
      <div>
        <Label htmlFor="name">이름</Label>
        <Input id="name" name="name" />
        {state.fieldErrors?.name && (
          <p className="text-sm text-destructive">{state.fieldErrors.name[0]}</p>
        )}
      </div>
      {state.error && (
        <Alert variant="destructive"><AlertDescription>{state.error}</AlertDescription></Alert>
      )}
      <SubmitButton />
    </form>
  )
}

function SubmitButton() {
  const { pending } = useFormStatus()
  return (
    <Button type="submit" disabled={pending}>
      {pending ? '처리 중...' : '사용자 등록'}
    </Button>
  )
}
```

---

## 생성 파일 목록

| 파일 | 설명 |
|:---|:---|
| `actions/[domain].ts` | `'use server'` — create/update/delete Actions |
| `components/[Domain]Form.tsx` | CC 폼 (`useFormState` + `useFormStatus`) |

---

## 산출물

- `actions/**/*.ts` — Server Actions 파일
- `components/**/*Form.tsx` — CC 폼 컴포넌트 (선택)
