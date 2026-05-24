# React 컴포넌트 코드 템플릿

## TypeScript 타입 정의 (`src/types/user.types.ts`)

```typescript
// 도메인 VO (API 응답 형식)
export interface UserVO {
  userId: number
  userNm: string
  email: string
  deptCd: string
  useYn: 'Y' | 'N'
  createdAt: string
}

// 목록 조회 필터
export interface UserFilter {
  userNm?: string
  deptCd?: string
  page?: number
  size?: number
}

// 등록 요청
export interface UserCreateReq {
  userNm: string
  email: string
  deptCd: string
}

// 수정 요청
export interface UserUpdateReq {
  userNm: string
  email: string
  deptCd: string
}

// API 공통 응답 래퍼
export interface ApiResponse<T> {
  code: string
  message: string
  data: T
}

// 페이지 응답
export interface PageResponse<T> {
  content: T[]
  totalElements: number
  totalPages: number
  page: number
  size: number
}
```

## Zod 스키마 (`src/types/user.schema.ts`)

```typescript
import { z } from 'zod'

export const userCreateSchema = z.object({
  userNm: z
    .string()
    .min(1, '사용자명을 입력하세요.')
    .max(50, '사용자명은 50자 이하입니다.'),
  email: z
    .string()
    .min(1, '이메일을 입력하세요.')
    .email('이메일 형식이 올바르지 않습니다.'),
  deptCd: z.string().min(1, '부서를 선택하세요.'),
})

export const userUpdateSchema = userCreateSchema

export type UserCreateForm = z.infer<typeof userCreateSchema>
export type UserUpdateForm = z.infer<typeof userUpdateSchema>
```

## API 클라이언트 함수 (`src/api/user.api.ts`)

```typescript
import apiClient from '@/lib/axios'
import type { ApiResponse, PageResponse, UserVO, UserFilter, UserCreateReq, UserUpdateReq } from '@/types/user.types'

export const userApi = {
  getList: (filter: UserFilter) =>
    apiClient
      .get<ApiResponse<PageResponse<UserVO>>>('/users', { params: filter })
      .then((r) => r.data.data),

  getOne: (id: number) =>
    apiClient
      .get<ApiResponse<UserVO>>(`/users/${id}`)
      .then((r) => r.data.data),

  create: (data: UserCreateReq) =>
    apiClient
      .post<ApiResponse<UserVO>>('/users', data)
      .then((r) => r.data.data),

  update: (id: number, data: UserUpdateReq) =>
    apiClient
      .put<ApiResponse<UserVO>>(`/users/${id}`, data)
      .then((r) => r.data.data),

  remove: (id: number) =>
    apiClient.delete(`/users/${id}`),
}
```

## queryKey 정의 (`src/api/queryKeys.ts`)

```typescript
import type { UserFilter } from '@/types/user.types'

export const userKeys = {
  all: ['users'] as const,
  list: (filter?: UserFilter) => [...userKeys.all, 'list', filter] as const,
  detail: (id: number) => [...userKeys.all, 'detail', id] as const,
}
```

## Custom Hook — 목록 조회 (`src/hooks/useUserList.ts`)

```typescript
import { useQuery } from '@tanstack/react-query'
import { userApi } from '@/api/user.api'
import { userKeys } from '@/api/queryKeys'
import type { UserFilter } from '@/types/user.types'

export function useUserList(filter: UserFilter) {
  return useQuery({
    queryKey: userKeys.list(filter),
    queryFn: () => userApi.getList(filter),
  })
}
```

## Custom Hook — 등록 뮤테이션 (`src/hooks/useCreateUser.ts`)

```typescript
import { useMutation, useQueryClient } from '@tanstack/react-query'
import { userApi } from '@/api/user.api'
import { userKeys } from '@/api/queryKeys'
import type { UserCreateReq } from '@/types/user.types'

export function useCreateUser() {
  const queryClient = useQueryClient()
  return useMutation({
    mutationFn: (data: UserCreateReq) => userApi.create(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: userKeys.all })
    },
  })
}
```

## 목록 페이지 (`src/pages/UserListPage.tsx`)

```typescript
import { useState } from 'react'
import { useNavigate } from 'react-router-dom'
import { useUserList } from '@/hooks/useUserList'
import { useDeleteUser } from '@/hooks/useDeleteUser'
import DataTable from '@/components/DataTable'
import Spinner from '@/components/Spinner'
import { Button } from '@/components/ui/button'
import type { UserVO, UserFilter } from '@/types/user.types'

export default function UserListPage() {
  const navigate = useNavigate()
  const [filter, setFilter] = useState<UserFilter>({ page: 0, size: 10 })
  const { data, isLoading, isError } = useUserList(filter)
  const deleteMutation = useDeleteUser()

  if (isLoading) return <Spinner />
  if (isError) return <p className="text-destructive">데이터를 불러오지 못했습니다.</p>

  const columns = [
    { key: 'userId', label: 'ID', width: 80 },
    { key: 'userNm', label: '사용자명' },
    { key: 'email', label: '이메일' },
    { key: 'deptCd', label: '부서코드' },
  ]

  return (
    <div className="p-6 space-y-4">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">사용자 목록</h1>
        <Button onClick={() => navigate('/users/create')} data-testid="btn-create">
          등록
        </Button>
      </div>

      <DataTable<UserVO>
        data={data?.content ?? []}
        columns={columns}
        totalCount={data?.totalElements ?? 0}
        onDelete={(row) => deleteMutation.mutate(row.userId)}
        onEdit={(row) => navigate(`/users/${row.userId}/edit`)}
        onDetail={(row) => navigate(`/users/${row.userId}`)}
      />
    </div>
  )
}
```

## 등록 폼 페이지 (`src/pages/UserCreatePage.tsx`)

```typescript
import { useNavigate } from 'react-router-dom'
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { useCreateUser } from '@/hooks/useCreateUser'
import { userCreateSchema, type UserCreateForm } from '@/types/user.schema'
import { Button } from '@/components/ui/button'
import { Input } from '@/components/ui/input'
import { Label } from '@/components/ui/label'

export default function UserCreatePage() {
  const navigate = useNavigate()
  const createMutation = useCreateUser()

  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<UserCreateForm>({
    resolver: zodResolver(userCreateSchema),
  })

  const onSubmit = (data: UserCreateForm) => {
    createMutation.mutate(data, {
      onSuccess: () => navigate('/users'),
    })
  }

  return (
    <div className="p-6 max-w-md space-y-6">
      <h1 className="text-2xl font-bold">사용자 등록</h1>
      <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
        <div className="space-y-1">
          <Label htmlFor="userNm">사용자명 *</Label>
          <Input
            id="userNm"
            data-testid="input-user-nm"
            {...register('userNm')}
          />
          {errors.userNm && (
            <p className="text-sm text-destructive">{errors.userNm.message}</p>
          )}
        </div>

        <div className="space-y-1">
          <Label htmlFor="email">이메일 *</Label>
          <Input
            id="email"
            type="email"
            data-testid="input-email"
            {...register('email')}
          />
          {errors.email && (
            <p className="text-sm text-destructive">{errors.email.message}</p>
          )}
        </div>

        <div className="flex gap-2 pt-2">
          <Button type="submit" disabled={createMutation.isPending} data-testid="btn-submit">
            {createMutation.isPending ? '등록 중...' : '등록'}
          </Button>
          <Button type="button" variant="outline" onClick={() => navigate(-1)}>
            취소
          </Button>
        </div>
      </form>
    </div>
  )
}
```

## 공통 DataTable 컴포넌트 (`src/components/DataTable.tsx`)

```typescript
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table'
import { Button } from '@/components/ui/button'

interface Column<T> {
  key: keyof T
  label: string
  width?: number
}

interface DataTableProps<T extends { [key: string]: unknown }> {
  data: T[]
  columns: Column<T>[]
  totalCount?: number
  onDetail?: (row: T) => void
  onEdit?: (row: T) => void
  onDelete?: (row: T) => void
}

export default function DataTable<T extends { [key: string]: unknown }>({
  data,
  columns,
  onDetail,
  onEdit,
  onDelete,
}: DataTableProps<T>) {
  return (
    <Table>
      <TableHeader>
        <TableRow>
          {columns.map((col) => (
            <TableHead key={String(col.key)} style={{ width: col.width }}>
              {col.label}
            </TableHead>
          ))}
          <TableHead>작업</TableHead>
        </TableRow>
      </TableHeader>
      <TableBody>
        {data.length === 0 ? (
          <TableRow>
            <TableCell colSpan={columns.length + 1} className="text-center text-muted-foreground">
              조회된 데이터가 없습니다.
            </TableCell>
          </TableRow>
        ) : (
          data.map((row, idx) => (
            <TableRow key={idx}>
              {columns.map((col) => (
                <TableCell key={String(col.key)}>{String(row[col.key] ?? '')}</TableCell>
              ))}
              <TableCell className="flex gap-1">
                {onDetail && (
                  <Button size="sm" variant="ghost" onClick={() => onDetail(row)}>
                    상세
                  </Button>
                )}
                {onEdit && (
                  <Button size="sm" variant="outline" onClick={() => onEdit(row)}>
                    수정
                  </Button>
                )}
                {onDelete && (
                  <Button
                    size="sm"
                    variant="destructive"
                    onClick={() => onDelete(row)}
                  >
                    삭제
                  </Button>
                )}
              </TableCell>
            </TableRow>
          ))
        )}
      </TableBody>
    </Table>
  )
}
```
