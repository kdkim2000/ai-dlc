---
name: ai-dlc-fe-shadcn-guide
description: AI-DLC 개발단계(프론트엔드-React) 스킬. shadcn/ui 컴포넌트 활용 가이드를 제공한다. "shadcn 컴포넌트 사용법", "shadcn/ui 가이드", "UI 컴포넌트 어떻게 써", "shadcn 설치 방법", "shadcn 버튼 사용법", "shadcn 다이얼로그", "radix ui 가이드" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC shadcn/ui 컴포넌트 활용 가이드

shadcn/ui(Radix UI 기반) 컴포넌트의 설치·초기화·주요 컴포넌트 사용법을 대화창에 출력한다. 파일을 수정하지 않는다.

## 트리거

- "shadcn 컴포넌트 사용법", "shadcn/ui 가이드", "UI 컴포넌트 어떻게 써"
- "shadcn 설치 방법", "shadcn 버튼 사용법", "shadcn 다이얼로그"
- "radix ui 가이드", "shadcn Form 연동", "shadcn 테마 설정"

---

## 설치 및 초기화

```bash
# shadcn/ui 초기화 (ai-dlc-fe-project-setup 완료 후)
npx shadcn@latest init

# 컴포넌트 개별 추가
npx shadcn@latest add button
npx shadcn@latest add dialog
npx shadcn@latest add form
npx shadcn@latest add table
npx shadcn@latest add select
npx shadcn@latest add input
npx shadcn@latest add toast
npx shadcn@latest add badge
npx shadcn@latest add card
```

---

## 주요 컴포넌트 사용법

### Button
```tsx
import { Button } from '@/components/ui/button'

// 기본
<Button>저장</Button>
<Button variant="outline">취소</Button>
<Button variant="destructive">삭제</Button>
<Button disabled={isLoading}>
  {isLoading ? '처리 중...' : '저장'}
</Button>

// 아이콘 버튼 (a11y: aria-label 필수)
<Button size="icon" aria-label="삭제">
  <Trash2 className="h-4 w-4" />
</Button>
```

### Dialog (모달)
```tsx
import {
  Dialog, DialogContent, DialogHeader,
  DialogTitle, DialogFooter,
} from '@/components/ui/dialog'

<Dialog open={open} onOpenChange={setOpen}>
  <DialogContent>
    <DialogHeader>
      <DialogTitle>사용자 삭제</DialogTitle>
    </DialogHeader>
    <p>정말 삭제하시겠습니까?</p>
    <DialogFooter>
      <Button variant="outline" onClick={() => setOpen(false)}>취소</Button>
      <Button variant="destructive" onClick={handleDelete}>삭제</Button>
    </DialogFooter>
  </DialogContent>
</Dialog>
```

### Form + Zod + React Hook Form
```tsx
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { Form, FormField, FormItem, FormLabel, FormControl, FormMessage } from '@/components/ui/form'
import { Input } from '@/components/ui/input'
import { userCreateSchema, type UserCreateReq } from '@/types/user.schema'

function UserCreateForm({ onSubmit }: { onSubmit: (data: UserCreateReq) => void }) {
  const form = useForm<UserCreateReq>({
    resolver: zodResolver(userCreateSchema),
    defaultValues: { userNm: '', email: '' },
  })

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="userNm"
          render={({ field }) => (
            <FormItem>
              <FormLabel>사용자명</FormLabel>
              <FormControl>
                <Input placeholder="사용자명 입력" data-testid="input-user-nm" {...field} />
              </FormControl>
              <FormMessage />   {/* Zod 에러 메시지 자동 표시 */}
            </FormItem>
          )}
        />
        <Button type="submit" data-testid="btn-submit">저장</Button>
      </form>
    </Form>
  )
}
```

### Table
```tsx
import {
  Table, TableBody, TableCell,
  TableHead, TableHeader, TableRow,
} from '@/components/ui/table'

<Table>
  <TableHeader>
    <TableRow>
      <TableHead>사용자명</TableHead>
      <TableHead>이메일</TableHead>
      <TableHead className="text-right">작업</TableHead>
    </TableRow>
  </TableHeader>
  <TableBody>
    {users.map((user) => (
      <TableRow key={user.userId}>
        <TableCell>{user.userNm}</TableCell>
        <TableCell>{user.email}</TableCell>
        <TableCell className="text-right">
          <Button size="sm" variant="outline">수정</Button>
        </TableCell>
      </TableRow>
    ))}
  </TableBody>
</Table>
```

### Select
```tsx
import { Select, SelectContent, SelectItem, SelectTrigger, SelectValue } from '@/components/ui/select'

<Select value={status} onValueChange={setStatus}>
  <SelectTrigger data-testid="select-status">
    <SelectValue placeholder="상태 선택" />
  </SelectTrigger>
  <SelectContent>
    <SelectItem value="ACTIVE">활성</SelectItem>
    <SelectItem value="INACTIVE">비활성</SelectItem>
  </SelectContent>
</Select>
```

### Toast (알림)
```tsx
import { useToast } from '@/hooks/use-toast'

function MyComponent() {
  const { toast } = useToast()

  const handleSuccess = () => {
    toast({ title: '저장 완료', description: '변경사항이 저장되었습니다.' })
  }

  const handleError = () => {
    toast({ title: '오류', description: '저장에 실패했습니다.', variant: 'destructive' })
  }
}
```

---

## 테마 커스터마이징 (CSS 변수)

```css
/* src/index.css — CSS 변수로 브랜드 색상 적용 */
:root {
  --primary: 221 83% 53%;        /* 브랜드 주색상 (HSL) */
  --primary-foreground: 0 0% 100%;
  --destructive: 0 72% 51%;
  --radius: 0.5rem;
}
```

---

## cn() 유틸리티로 클래스 조합

```typescript
// src/utils/cn.ts
import { clsx, type ClassValue } from 'clsx'
import { twMerge } from 'tailwind-merge'

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs))
}

// 사용
<div className={cn('base-class', isActive && 'active-class', className)} />
```

---

## 컴포넌트 확장 패턴

```tsx
// shadcn 컴포넌트를 래핑하여 프로젝트 전용 컴포넌트 생성
interface DataTableProps<T> {
  data: T[]
  columns: Column<T>[]
  isLoading?: boolean
  emptyMessage?: string
}

export function DataTable<T>({ data, columns, isLoading, emptyMessage = '데이터가 없습니다.' }: DataTableProps<T>) {
  if (isLoading) return <div className="flex justify-center p-8"><LoadingSpinner /></div>
  if (data.length === 0) return <p className="text-center text-muted-foreground py-8">{emptyMessage}</p>
  // ... Table 렌더링
}
```
