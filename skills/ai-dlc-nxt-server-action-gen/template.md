# Next.js Server Actions 코드 템플릿

---

## actions/user.ts

```typescript
'use server'

import { z } from 'zod'
import { revalidatePath, revalidateTag } from 'next/cache'
import { redirect } from 'next/navigation'
import { auth } from '@/lib/auth'
import { prisma } from '@/lib/prisma'

// ─── 공통 타입 ────────────────────────────────────────────────────────────────

export type FormState = {
  error: string | null
  fieldErrors?: Record<string, string[]>
  success?: boolean
}

// ─── Zod 스키마 ──────────────────────────────────────────────────────────────

const CreateUserSchema = z.object({
  name: z.string().min(2, '이름은 2자 이상이어야 합니다'),
  email: z.string().email('유효한 이메일 형식을 입력하세요'),
  role: z.enum(['ADMIN', 'USER'], { message: '역할을 선택하세요' }),
})

const UpdateUserSchema = CreateUserSchema.partial().extend({
  id: z.string().min(1),
})

// ─── Server Actions ───────────────────────────────────────────────────────────

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
  } catch (error: unknown) {
    if (
      typeof error === 'object' &&
      error !== null &&
      'code' in error &&
      (error as { code: string }).code === 'P2002'
    ) {
      return { error: '이미 사용 중인 이메일입니다' }
    }
    return { error: '사용자 등록에 실패했습니다' }
  }

  revalidatePath('/users')
  redirect('/users')
}

export async function updateUser(
  _prevState: FormState,
  formData: FormData,
): Promise<FormState> {
  const session = await auth()
  if (!session) return { error: '인증이 필요합니다' }

  const parsed = UpdateUserSchema.safeParse({
    id: formData.get('id'),
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

  const { id, ...data } = parsed.data

  try {
    await prisma.user.update({ where: { id }, data })
  } catch (error: unknown) {
    if (
      typeof error === 'object' &&
      error !== null &&
      'code' in error &&
      (error as { code: string }).code === 'P2025'
    ) {
      return { error: '사용자를 찾을 수 없습니다' }
    }
    return { error: '사용자 수정에 실패했습니다' }
  }

  revalidatePath('/users')
  revalidatePath(`/users/${id}`)
  redirect(`/users/${id}`)
}

export async function deleteUser(userId: string): Promise<FormState> {
  const session = await auth()
  if (!session) return { error: '인증이 필요합니다' }

  try {
    await prisma.user.delete({ where: { id: userId } })
  } catch {
    return { error: '사용자 삭제에 실패했습니다' }
  }

  revalidatePath('/users')
  return { error: null, success: true }
}
```

---

## components/CreateUserForm.tsx (CC 연동 폼)

```typescript
'use client'

import { useFormState, useFormStatus } from 'react-dom'
import { createUser, type FormState } from '@/actions/user'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select'
import { Alert, AlertDescription } from '@/components/ui/alert'

const initialState: FormState = { error: null }

export function CreateUserForm() {
  const [state, action] = useFormState(createUser, initialState)

  return (
    <form action={action} className="space-y-4 max-w-md">
      <div className="space-y-2">
        <Label htmlFor="name">이름</Label>
        <Input id="name" name="name" placeholder="홍길동" />
        {state.fieldErrors?.name && (
          <p className="text-sm text-destructive">{state.fieldErrors.name[0]}</p>
        )}
      </div>

      <div className="space-y-2">
        <Label htmlFor="email">이메일</Label>
        <Input id="email" name="email" type="email" placeholder="user@example.com" />
        {state.fieldErrors?.email && (
          <p className="text-sm text-destructive">{state.fieldErrors.email[0]}</p>
        )}
      </div>

      <div className="space-y-2">
        <Label htmlFor="role">역할</Label>
        <Select name="role" defaultValue="USER">
          <SelectTrigger>
            <SelectValue placeholder="역할 선택" />
          </SelectTrigger>
          <SelectContent>
            <SelectItem value="USER">일반 사용자</SelectItem>
            <SelectItem value="ADMIN">관리자</SelectItem>
          </SelectContent>
        </Select>
        {state.fieldErrors?.role && (
          <p className="text-sm text-destructive">{state.fieldErrors.role[0]}</p>
        )}
      </div>

      {state.error && (
        <Alert variant="destructive">
          <AlertDescription>{state.error}</AlertDescription>
        </Alert>
      )}

      <SubmitButton />
    </form>
  )
}

function SubmitButton() {
  const { pending } = useFormStatus()
  return (
    <Button type="submit" disabled={pending} className="w-full">
      {pending ? '등록 중...' : '사용자 등록'}
    </Button>
  )
}
```

---

## components/EditUserForm.tsx (수정 폼 — defaultValue 주입)

```typescript
'use client'

import { useFormState, useFormStatus } from 'react-dom'
import { updateUser, type FormState } from '@/actions/user'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'
import { Alert, AlertDescription } from '@/components/ui/alert'
import type { User } from '@/types/user'

const initialState: FormState = { error: null }

interface EditUserFormProps {
  user: User
}

export function EditUserForm({ user }: EditUserFormProps) {
  const [state, action] = useFormState(updateUser, initialState)

  return (
    <form action={action} className="space-y-4 max-w-md">
      <input type="hidden" name="id" value={user.id} />

      <div className="space-y-2">
        <Label htmlFor="name">이름</Label>
        <Input id="name" name="name" defaultValue={user.name} />
        {state.fieldErrors?.name && (
          <p className="text-sm text-destructive">{state.fieldErrors.name[0]}</p>
        )}
      </div>

      <div className="space-y-2">
        <Label htmlFor="email">이메일</Label>
        <Input id="email" name="email" type="email" defaultValue={user.email} />
        {state.fieldErrors?.email && (
          <p className="text-sm text-destructive">{state.fieldErrors.email[0]}</p>
        )}
      </div>

      {state.error && (
        <Alert variant="destructive">
          <AlertDescription>{state.error}</AlertDescription>
        </Alert>
      )}

      <SubmitButton />
    </form>
  )
}

function SubmitButton() {
  const { pending } = useFormStatus()
  return (
    <Button type="submit" disabled={pending} className="w-full">
      {pending ? '저장 중...' : '저장'}
    </Button>
  )
}
```
