# Vue SFC 컴포넌트 코드 패턴 템플릿

## src/views/UsersView.vue (List View — useQuery)

```vue
<script setup lang="ts">
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { useUserList } from '@/composables/useUserList'
import UserTable from '@/components/UserTable.vue'
import SearchForm from '@/components/SearchForm.vue'
import LoadingSpinner from '@/components/LoadingSpinner.vue'
import ErrorMessage from '@/components/ErrorMessage.vue'
import { Button } from '@/components/ui/button'

const router = useRouter()
const { users, totalCount, isLoading, isError, error, filters } = useUserList()

function handleSearch(newFilters: typeof filters.value) {
  filters.value = { ...newFilters, page: 1 }
}

function handlePageChange(page: number) {
  filters.value.page = page
}

function goToCreate() {
  router.push('/users/new')
}
</script>

<template>
  <div class="container py-6">
    <div class="flex items-center justify-between mb-6">
      <h1 class="text-2xl font-bold">사용자 관리</h1>
      <Button @click="goToCreate" data-testid="btn-create">
        사용자 등록
      </Button>
    </div>

    <SearchForm class="mb-4" @search="handleSearch" />

    <LoadingSpinner v-if="isLoading" />
    <ErrorMessage v-else-if="isError" :message="error?.message" />
    <template v-else>
      <UserTable
        :users="users"
        data-testid="user-table"
      />
      <div class="mt-4 text-sm text-muted-foreground">
        총 {{ totalCount }}건
      </div>
    </template>
  </div>
</template>
```

---

## src/components/UserTable.vue (Presentational)

```vue
<script setup lang="ts">
import { useRouter } from 'vue-router'
import type { User } from '@/types/user.types'
import { Button } from '@/components/ui/button'
import ConfirmDialog from '@/components/ConfirmDialog.vue'
import { ref } from 'vue'

const props = defineProps<{
  users: User[]
}>()

const emit = defineEmits<{
  (e: 'delete', id: number): void
}>()

const router = useRouter()
const deleteTargetId = ref<number | null>(null)

function goToDetail(id: number) {
  router.push(`/users/${id}`)
}

function goToEdit(id: number) {
  router.push(`/users/${id}/edit`)
}

function confirmDelete(id: number) {
  deleteTargetId.value = id
}

function handleDeleteConfirm() {
  if (deleteTargetId.value !== null) {
    emit('delete', deleteTargetId.value)
    deleteTargetId.value = null
  }
}
</script>

<template>
  <div class="rounded-md border">
    <table class="w-full text-sm">
      <thead>
        <tr class="border-b bg-muted/50">
          <th class="px-4 py-3 text-left font-medium">이름</th>
          <th class="px-4 py-3 text-left font-medium">이메일</th>
          <th class="px-4 py-3 text-left font-medium">역할</th>
          <th class="px-4 py-3 text-left font-medium">작업</th>
        </tr>
      </thead>
      <tbody>
        <tr
          v-for="user in users"
          :key="user.id"
          class="border-b last:border-0 hover:bg-muted/30"
        >
          <td class="px-4 py-3">{{ user.name }}</td>
          <td class="px-4 py-3">{{ user.email }}</td>
          <td class="px-4 py-3">{{ user.role }}</td>
          <td class="px-4 py-3 space-x-2">
            <Button variant="ghost" size="sm" @click="goToDetail(user.id)">
              상세
            </Button>
            <Button variant="ghost" size="sm" @click="goToEdit(user.id)">
              수정
            </Button>
            <Button
              variant="ghost"
              size="sm"
              class="text-destructive"
              @click="confirmDelete(user.id)"
              :data-testid="`btn-delete-${user.id}`"
            >
              삭제
            </Button>
          </td>
        </tr>
        <tr v-if="users.length === 0">
          <td colspan="4" class="px-4 py-8 text-center text-muted-foreground">
            데이터가 없습니다.
          </td>
        </tr>
      </tbody>
    </table>
  </div>

  <ConfirmDialog
    :open="deleteTargetId !== null"
    title="사용자 삭제"
    description="삭제한 사용자는 복구할 수 없습니다. 계속하시겠습니까?"
    @confirm="handleDeleteConfirm"
    @cancel="deleteTargetId = null"
  />
</template>
```

---

## src/components/UserForm.vue (VeeValidate + Zod)

```vue
<script setup lang="ts">
import { useForm, useField } from 'vee-validate'
import { toTypedSchema } from '@vee-validate/zod'
import { z } from 'zod'
import type { CreateUserDto } from '@/types/user.types'
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

const props = withDefaults(
  defineProps<{
    defaultValues?: Partial<CreateUserDto>
    isSubmitting?: boolean
  }>(),
  { isSubmitting: false },
)

const emit = defineEmits<{
  (e: 'submit', values: CreateUserDto): void
  (e: 'cancel'): void
}>()

const schema = z.object({
  name: z.string().min(1, '이름을 입력하세요').max(50, '이름은 50자 이내로 입력하세요'),
  email: z.string().email('올바른 이메일 형식이 아닙니다'),
  role: z.enum(['ADMIN', 'USER'], { required_error: '역할을 선택하세요' }),
})

const { handleSubmit, errors } = useForm({
  validationSchema: toTypedSchema(schema),
  initialValues: props.defaultValues,
})

const { value: name } = useField<string>('name')
const { value: email } = useField<string>('email')
const { value: role } = useField<string>('role')

const onSubmit = handleSubmit((values) => {
  emit('submit', values as CreateUserDto)
})
</script>

<template>
  <form @submit="onSubmit" class="space-y-4" data-testid="user-form">
    <div class="space-y-2">
      <Label for="name">이름 <span class="text-destructive">*</span></Label>
      <Input
        id="name"
        v-model="name"
        placeholder="이름을 입력하세요"
        data-testid="input-name"
      />
      <p v-if="errors.name" class="text-sm text-destructive" role="alert">
        {{ errors.name }}
      </p>
    </div>

    <div class="space-y-2">
      <Label for="email">이메일 <span class="text-destructive">*</span></Label>
      <Input
        id="email"
        v-model="email"
        type="email"
        placeholder="이메일을 입력하세요"
        data-testid="input-email"
      />
      <p v-if="errors.email" class="text-sm text-destructive" role="alert">
        {{ errors.email }}
      </p>
    </div>

    <div class="space-y-2">
      <Label>역할 <span class="text-destructive">*</span></Label>
      <Select v-model="role" data-testid="select-role">
        <SelectTrigger>
          <SelectValue placeholder="역할을 선택하세요" />
        </SelectTrigger>
        <SelectContent>
          <SelectItem value="USER">일반 사용자</SelectItem>
          <SelectItem value="ADMIN">관리자</SelectItem>
        </SelectContent>
      </Select>
      <p v-if="errors.role" class="text-sm text-destructive" role="alert">
        {{ errors.role }}
      </p>
    </div>

    <div class="flex justify-end gap-2 pt-2">
      <Button type="button" variant="outline" @click="emit('cancel')">
        취소
      </Button>
      <Button type="submit" :disabled="isSubmitting" data-testid="btn-submit">
        {{ isSubmitting ? '저장 중...' : '저장' }}
      </Button>
    </div>
  </form>
</template>
```

---

## src/composables/useUsers.ts

```typescript
import { ref, computed } from 'vue'
import { useQuery, useMutation, useQueryClient } from '@tanstack/vue-query'
import { useRouter } from 'vue-router'
import { fetchUserList, fetchUserById, createUser, updateUser, deleteUser } from '@/api/user.api'
import type { UserFilters, CreateUserDto, UpdateUserDto } from '@/types/user.types'

// 목록 조회 Composable
export function useUserList() {
  const filters = ref<UserFilters>({ page: 1, size: 20 })

  const { data, isLoading, isError, error } = useQuery({
    queryKey: computed(() => ['users', 'list', filters.value]),
    queryFn: () => fetchUserList(filters.value),
  })

  const users = computed(() => data.value?.content ?? [])
  const totalCount = computed(() => data.value?.totalElements ?? 0)

  return { users, totalCount, isLoading, isError, error, filters }
}

// 단건 조회 + 삭제 Composable
export function useUserDetail(userId: Ref<number>) {
  const router = useRouter()
  const queryClient = useQueryClient()

  const { data: user, isLoading } = useQuery({
    queryKey: computed(() => ['users', 'detail', userId.value]),
    queryFn: () => fetchUserById(userId.value),
    enabled: computed(() => !!userId.value),
  })

  const { mutate: handleDelete, isPending: isDeleting } = useMutation({
    mutationFn: () => deleteUser(userId.value),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users', 'list'] })
      router.push('/users')
    },
  })

  return { user, isLoading, handleDelete, isDeleting }
}

// 폼 (생성/수정) Composable
export function useUserForm(userId?: Ref<number | undefined>) {
  const router = useRouter()
  const queryClient = useQueryClient()

  const { mutate: submitCreate, isPending: isCreating } = useMutation({
    mutationFn: (data: CreateUserDto) => createUser(data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users', 'list'] })
      router.push('/users')
    },
  })

  const { mutate: submitUpdate, isPending: isUpdating } = useMutation({
    mutationFn: (data: UpdateUserDto) => updateUser(userId!.value!, data),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ['users'] })
      router.push(`/users/${userId!.value}`)
    },
  })

  const isSubmitting = computed(() => isCreating.value || isUpdating.value)

  return { submitCreate, submitUpdate, isSubmitting }
}
```

---

## src/stores/userStore.ts (Pinia — UI 상태만)

```typescript
import { defineStore } from 'pinia'
import { ref } from 'vue'

export const useUserStore = defineStore('user', () => {
  // UI 상태만 Pinia에 저장; 서버 데이터는 Vue Query 담당
  const selectedUserId = ref<number | null>(null)
  const isDeleteDialogOpen = ref(false)

  function openDeleteDialog(id: number) {
    selectedUserId.value = id
    isDeleteDialogOpen.value = true
  }

  function closeDeleteDialog() {
    selectedUserId.value = null
    isDeleteDialogOpen.value = false
  }

  return { selectedUserId, isDeleteDialogOpen, openDeleteDialog, closeDeleteDialog }
})
```

---

## src/api/user.api.ts

```typescript
import { axiosInstance } from '@/lib/axios'
import type {
  User,
  UserFilters,
  PageResponse,
  CreateUserDto,
  UpdateUserDto,
} from '@/types/user.types'

export async function fetchUserList(filters: UserFilters): Promise<PageResponse<User>> {
  const { data } = await axiosInstance.get<PageResponse<User>>('/api/users', {
    params: filters,
  })
  return data
}

export async function fetchUserById(id: number): Promise<User> {
  const { data } = await axiosInstance.get<User>(`/api/users/${id}`)
  return data
}

export async function createUser(dto: CreateUserDto): Promise<User> {
  const { data } = await axiosInstance.post<User>('/api/users', dto)
  return data
}

export async function updateUser(id: number, dto: UpdateUserDto): Promise<User> {
  const { data } = await axiosInstance.put<User>(`/api/users/${id}`, dto)
  return data
}

export async function deleteUser(id: number): Promise<void> {
  await axiosInstance.delete(`/api/users/${id}`)
}
```

---

## src/types/user.types.ts

```typescript
export interface User {
  id: number
  name: string
  email: string
  role: 'ADMIN' | 'USER'
  createdAt: string
}

export interface UserFilters {
  page: number
  size: number
  name?: string
  email?: string
  role?: 'ADMIN' | 'USER'
}

export interface CreateUserDto {
  name: string
  email: string
  role: 'ADMIN' | 'USER'
}

export type UpdateUserDto = Partial<CreateUserDto>

export interface PageResponse<T> {
  content: T[]
  totalElements: number
  totalPages: number
  number: number
  size: number
}
```
