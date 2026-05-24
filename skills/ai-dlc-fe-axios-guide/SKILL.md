---
name: ai-dlc-fe-axios-guide
description: AI-DLC 개발단계(프론트엔드-React) 스킬. Axios HTTP 클라이언트 활용 가이드를 제공한다. "Axios 가이드", "HTTP 클라이언트 사용법", "API 연동 방법", "Axios 인스턴스 설정", "Axios 인터셉터", "토큰 자동 주입", "Axios 에러 처리" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Axios HTTP 클라이언트 활용 가이드

Axios 인스턴스 생성·인터셉터 설정·TypeScript 연동·에러 처리·React Query 연동 패턴을 대화창에 출력한다. 파일을 수정하지 않는다.

## 트리거

- "Axios 가이드", "HTTP 클라이언트 사용법", "API 연동 방법"
- "Axios 인스턴스 설정", "Axios 인터셉터", "토큰 자동 주입"
- "Axios 에러 처리", "AxiosError 타입", "API 공통 설정"

---

## Axios 인스턴스 설정 (`src/lib/axios.ts`)

```typescript
import axios, { type AxiosError, type AxiosResponse } from 'axios'

export const apiClient = axios.create({
  baseURL: import.meta.env.VITE_API_BASE_URL ?? 'http://localhost:8080',
  timeout: 10_000,
  headers: { 'Content-Type': 'application/json' },
})

// 요청 인터셉터 — Access Token 자동 주입
apiClient.interceptors.request.use((config) => {
  const token = localStorage.getItem('accessToken')
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

// 응답 인터셉터 — 에러 공통 처리
apiClient.interceptors.response.use(
  (response: AxiosResponse) => response,
  (error: AxiosError<ApiErrorBody>) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('accessToken')
      window.location.href = '/login'
    }
    return Promise.reject(error)
  }
)
```

---

## TypeScript 타입 정의

```typescript
// src/types/common.types.ts
export interface ApiResponse<T> {
  code: string
  message: string
  data: T
}

export interface PageResponse<T> {
  content: T[]
  totalElements: number
  totalPages: number
  page: number
  size: number
}

export interface ApiErrorBody {
  code: string
  message: string
  errors?: Array<{ field: string; message: string }>
}
```

---

## API 함수 모듈 (`src/api/user.api.ts`)

```typescript
import { apiClient } from '@/lib/axios'
import type { ApiResponse, PageResponse } from '@/types/common.types'
import type { UserVO, UserFilter, UserCreateReq, UserUpdateReq } from '@/types/user.types'

export const userApi = {
  getList: (params: UserFilter) =>
    apiClient.get<ApiResponse<PageResponse<UserVO>>>('/api/users', { params })
      .then((res) => res.data.data),

  getOne: (userId: number) =>
    apiClient.get<ApiResponse<UserVO>>(`/api/users/${userId}`)
      .then((res) => res.data.data),

  create: (body: UserCreateReq) =>
    apiClient.post<ApiResponse<UserVO>>('/api/users', body)
      .then((res) => res.data.data),

  update: (userId: number, body: UserUpdateReq) =>
    apiClient.put<ApiResponse<UserVO>>(`/api/users/${userId}`, body)
      .then((res) => res.data.data),

  remove: (userId: number) =>
    apiClient.delete<ApiResponse<void>>(`/api/users/${userId}`)
      .then((res) => res.data),
}
```

---

## AxiosError 타입 가드

```typescript
import { isAxiosError } from 'axios'
import type { ApiErrorBody } from '@/types/common.types'

export function getErrorMessage(error: unknown): string {
  if (isAxiosError<ApiErrorBody>(error)) {
    // 서버 에러 메시지
    return error.response?.data?.message ?? '서버 오류가 발생했습니다.'
  }
  if (error instanceof Error) {
    return error.message
  }
  return '알 수 없는 오류가 발생했습니다.'
}

// 사용 예시 (useMutation onError)
onError: (error) => {
  toast({ title: '오류', description: getErrorMessage(error), variant: 'destructive' })
}
```

---

## React Query와 연동 패턴

```typescript
// src/hooks/useUserList.ts
import { useQuery } from '@tanstack/react-query'
import { userApi } from '@/api/user.api'
import { userKeys } from '@/api/queryKeys'
import type { UserFilter } from '@/types/user.types'

export function useUserList(filters: UserFilter) {
  return useQuery({
    queryKey: userKeys.list(filters),
    queryFn: () => userApi.getList(filters),
    staleTime: 1000 * 60 * 5,  // 5분
  })
}

// src/hooks/useCreateUser.ts
import { useMutation, useQueryClient } from '@tanstack/react-query'
import { userApi } from '@/api/user.api'
import { userKeys } from '@/api/queryKeys'

export function useCreateUser() {
  const queryClient = useQueryClient()

  return useMutation({
    mutationFn: userApi.create,
    onSuccess: () => {
      // 목록 캐시 무효화 → 자동 재조회
      queryClient.invalidateQueries({ queryKey: userKeys.all })
    },
  })
}
```

---

## 파일 업로드

```typescript
export const fileApi = {
  upload: (file: File) => {
    const formData = new FormData()
    formData.append('file', file)
    return apiClient.post<ApiResponse<{ fileUrl: string }>>('/api/files', formData, {
      headers: { 'Content-Type': 'multipart/form-data' },
    }).then((res) => res.data.data)
  },
}
```

---

## 요청 취소 (AbortController)

```typescript
// React Query에서는 signal이 자동 전달됨
export const userApi = {
  getList: (params: UserFilter, signal?: AbortSignal) =>
    apiClient.get<ApiResponse<PageResponse<UserVO>>>('/api/users', { params, signal })
      .then((res) => res.data.data),
}

// useQuery에서 자동 취소 적용
useQuery({
  queryKey: userKeys.list(filters),
  queryFn: ({ signal }) => userApi.getList(filters, signal),
})
```

---

## 환경변수 설정

```
# .env
VITE_API_BASE_URL=http://localhost:8080

# .env.production
VITE_API_BASE_URL=https://api.example.com
```

> Vite 환경에서 `VITE_` 접두사 없는 변수는 클라이언트에 노출되지 않는다.
