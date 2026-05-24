---
name: ai-dlc-vue-ui-guide
description: AI-DLC 개발단계(프론트엔드-Vue.js) 가이드 스킬. shadcn-vue / radix-vue UI 컴포넌트 설정 및 사용법을 안내한다. "shadcn-vue 설치", "radix-vue 컴포넌트", "Vue UI 라이브러리 설정", "shadcn-vue 사용법", "Vue UI 컴포넌트", "다크모드 Vue" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC shadcn-vue / radix-vue UI 컴포넌트 가이드

Vue.js 프로젝트에서 shadcn-vue와 radix-vue를 사용하여 UI 컴포넌트를 구성하는 방법을 안내한다.

## 트리거

- "shadcn-vue 설치", "radix-vue 컴포넌트", "Vue UI 라이브러리 설정"
- "shadcn-vue 사용법", "Vue UI 컴포넌트", "다크모드 Vue"
- "Vue 버튼 컴포넌트", "Vue 다이얼로그", "Vue 테이블 컴포넌트"

---

## shadcn-vue 초기 설정

```bash
# 프로젝트에 shadcn-vue 초기화
npx shadcn-vue@latest init

# 컴포넌트 추가
npx shadcn-vue@latest add button
npx shadcn-vue@latest add input
npx shadcn-vue@latest add dialog
npx shadcn-vue@latest add select
npx shadcn-vue@latest add table
npx shadcn-vue@latest add card
npx shadcn-vue@latest add badge
npx shadcn-vue@latest add dropdown-menu
```

생성 위치: `src/components/ui/`

---

## Button

```vue
<script setup lang="ts">
import { Button } from '@/components/ui/button'
</script>

<template>
  <!-- 기본 -->
  <Button>클릭</Button>

  <!-- variant -->
  <Button variant="default">기본</Button>
  <Button variant="destructive">삭제</Button>
  <Button variant="outline">외곽선</Button>
  <Button variant="ghost">고스트</Button>
  <Button variant="link">링크</Button>

  <!-- size -->
  <Button size="sm">작게</Button>
  <Button size="default">기본</Button>
  <Button size="lg">크게</Button>
  <Button size="icon"><TrashIcon class="h-4 w-4" /></Button>

  <!-- 비활성화 -->
  <Button :disabled="isLoading">{{ isLoading ? '처리 중...' : '저장' }}</Button>

  <!-- 로딩 스피너 포함 -->
  <Button :disabled="isPending">
    <Loader2Icon v-if="isPending" class="mr-2 h-4 w-4 animate-spin" />
    {{ isPending ? '저장 중...' : '저장' }}
  </Button>
</template>
```

---

## Dialog (확인 다이얼로그)

```vue
<script setup lang="ts">
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogFooter,
  DialogHeader,
  DialogTitle,
} from '@/components/ui/dialog'
import { Button } from '@/components/ui/button'

const props = defineProps<{
  open: boolean
  title: string
  description: string
}>()

const emit = defineEmits<{
  (e: 'confirm'): void
  (e: 'cancel'): void
}>()
</script>

<template>
  <Dialog :open="open" @update:open="(val) => !val && emit('cancel')">
    <DialogContent>
      <DialogHeader>
        <DialogTitle>{{ title }}</DialogTitle>
        <DialogDescription>{{ description }}</DialogDescription>
      </DialogHeader>
      <DialogFooter>
        <Button variant="outline" @click="emit('cancel')">취소</Button>
        <Button variant="destructive" @click="emit('confirm')" data-testid="btn-confirm">
          확인
        </Button>
      </DialogFooter>
    </DialogContent>
  </Dialog>
</template>
```

---

## Select

```vue
<script setup lang="ts">
import {
  Select,
  SelectContent,
  SelectItem,
  SelectTrigger,
  SelectValue,
} from '@/components/ui/select'

const value = ref('')
</script>

<template>
  <Select v-model="value">
    <SelectTrigger>
      <SelectValue placeholder="항목을 선택하세요" />
    </SelectTrigger>
    <SelectContent>
      <SelectItem value="option1">옵션 1</SelectItem>
      <SelectItem value="option2">옵션 2</SelectItem>
    </SelectContent>
  </Select>
</template>
```

---

## Table

```vue
<script setup lang="ts">
import {
  Table,
  TableBody,
  TableCaption,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table'
</script>

<template>
  <Table>
    <TableCaption>사용자 목록</TableCaption>
    <TableHeader>
      <TableRow>
        <TableHead>이름</TableHead>
        <TableHead>이메일</TableHead>
        <TableHead class="text-right">작업</TableHead>
      </TableRow>
    </TableHeader>
    <TableBody>
      <TableRow v-for="user in users" :key="user.id">
        <TableCell>{{ user.name }}</TableCell>
        <TableCell>{{ user.email }}</TableCell>
        <TableCell class="text-right">
          <Button variant="ghost" size="sm">수정</Button>
        </TableCell>
      </TableRow>
      <TableRow v-if="users.length === 0">
        <TableCell colspan="3" class="text-center text-muted-foreground py-8">
          데이터가 없습니다.
        </TableCell>
      </TableRow>
    </TableBody>
  </Table>
</template>
```

---

## 다크모드 (useColorMode)

```bash
npm install @vueuse/core
```

```typescript
// src/composables/useTheme.ts
import { useColorMode, usePreferredDark } from '@vueuse/core'

export function useTheme() {
  const mode = useColorMode()  // 'light' | 'dark' | 'auto'
  const prefersDark = usePreferredDark()

  function toggleTheme() {
    mode.value = mode.value === 'dark' ? 'light' : 'dark'
  }

  return { mode, prefersDark, toggleTheme }
}
```

```vue
<template>
  <Button variant="ghost" size="icon" @click="toggleTheme">
    <SunIcon v-if="mode === 'dark'" class="h-5 w-5" />
    <MoonIcon v-else class="h-5 w-5" />
  </Button>
</template>
```

---

## Card

```vue
<template>
  <Card>
    <CardHeader>
      <CardTitle>카드 제목</CardTitle>
      <CardDescription>카드 설명</CardDescription>
    </CardHeader>
    <CardContent>
      <p>카드 내용</p>
    </CardContent>
    <CardFooter class="flex justify-end gap-2">
      <Button variant="outline">취소</Button>
      <Button>저장</Button>
    </CardFooter>
  </Card>
</template>
```

---

## Badge

```vue
<template>
  <Badge>기본</Badge>
  <Badge variant="secondary">보조</Badge>
  <Badge variant="destructive">삭제</Badge>
  <Badge variant="outline">외곽선</Badge>

  <!-- 동적 variant -->
  <Badge :variant="user.role === 'ADMIN' ? 'destructive' : 'secondary'">
    {{ user.role }}
  </Badge>
</template>
```

---

## Tailwind CSS 커스텀 테마

```typescript
// tailwind.config.ts
export default {
  theme: {
    extend: {
      colors: {
        // CSS 변수 기반 — shadcn-vue와 호환
        primary: { DEFAULT: 'hsl(var(--primary))', foreground: 'hsl(var(--primary-foreground))' },
        // 커스텀 색상 추가
        brand: {
          50: '#eff6ff',
          500: '#3b82f6',
          900: '#1e3a8a',
        },
      },
    },
  },
}
```
