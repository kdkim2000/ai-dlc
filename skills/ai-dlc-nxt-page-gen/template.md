# Next.js App Router 페이지·컴포넌트 코드 템플릿

---

## app/(dashboard)/users/page.tsx (RSC 목록 페이지)

```typescript
import { Suspense } from 'react'
import type { Metadata } from 'next'
import { UserTable } from '@/components/UserTable'
import { UserTableSkeleton } from '@/components/UserTableSkeleton'
import { Button } from '@/components/ui/button'
import Link from 'next/link'
import { auth } from '@/lib/auth'
import { redirect } from 'next/navigation'

export const metadata: Metadata = {
  title: '사용자 목록',
}

interface PageProps {
  searchParams: { page?: string; keyword?: string }
}

async function getUsers(searchParams: PageProps['searchParams']) {
  const params = new URLSearchParams({
    page: searchParams.page ?? '1',
    ...(searchParams.keyword && { keyword: searchParams.keyword }),
  })
  const res = await fetch(
    `${process.env.API_BASE_URL}/api/users?${params}`,
    { next: { revalidate: 60 } },
  )
  if (!res.ok) throw new Error('사용자 목록을 불러오는 데 실패했습니다')
  return res.json()
}

export default async function UsersPage({ searchParams }: PageProps) {
  const session = await auth()
  if (!session) redirect('/login')

  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">사용자 관리</h1>
        <Button asChild>
          <Link href="/users/new">사용자 등록</Link>
        </Button>
      </div>
      <Suspense fallback={<UserTableSkeleton />}>
        <UserTableWrapper searchParams={searchParams} />
      </Suspense>
    </div>
  )
}

async function UserTableWrapper({ searchParams }: PageProps) {
  const data = await getUsers(searchParams)
  return <UserTable users={data.data} pagination={data.pagination} />
}
```

---

## app/(dashboard)/users/[id]/page.tsx (RSC 상세 페이지)

```typescript
import type { Metadata } from 'next'
import { notFound } from 'next/navigation'
import { UserDetail } from '@/components/UserDetail'
import { UserDeleteButton } from '@/components/UserDeleteButton'
import Link from 'next/link'
import { Button } from '@/components/ui/button'

interface PageProps {
  params: { id: string }
}

async function getUser(id: string) {
  const res = await fetch(
    `${process.env.API_BASE_URL}/api/users/${id}`,
    { cache: 'force-cache' },
  )
  if (res.status === 404) return null
  if (!res.ok) throw new Error('사용자 정보를 불러오는 데 실패했습니다')
  return res.json()
}

export async function generateMetadata({ params }: PageProps): Promise<Metadata> {
  const data = await getUser(params.id)
  return { title: data ? `${data.data.name} 상세` : '사용자 없음' }
}

export default async function UserDetailPage({ params }: PageProps) {
  const data = await getUser(params.id)
  if (!data) notFound()

  return (
    <div className="space-y-6">
      <div className="flex items-center justify-between">
        <h1 className="text-2xl font-bold">사용자 상세</h1>
        <div className="flex gap-2">
          <Button variant="outline" asChild>
            <Link href={`/users/${params.id}/edit`}>수정</Link>
          </Button>
          <UserDeleteButton userId={params.id} />
        </div>
      </div>
      <UserDetail user={data.data} />
    </div>
  )
}
```

---

## app/(dashboard)/users/loading.tsx (스켈레톤)

```typescript
import { Skeleton } from '@/components/ui/skeleton'

export default function UsersLoading() {
  return (
    <div className="space-y-4">
      <div className="flex items-center justify-between">
        <Skeleton className="h-8 w-32" />
        <Skeleton className="h-10 w-24" />
      </div>
      <div className="rounded-md border">
        {Array.from({ length: 5 }).map((_, i) => (
          <div key={i} className="flex items-center gap-4 p-4 border-b last:border-0">
            <Skeleton className="h-4 w-8" />
            <Skeleton className="h-4 w-32" />
            <Skeleton className="h-4 w-48" />
            <Skeleton className="h-4 w-20" />
          </div>
        ))}
      </div>
    </div>
  )
}
```

---

## app/(dashboard)/users/error.tsx (에러 경계)

```typescript
'use client'

import { useEffect } from 'react'
import { Button } from '@/components/ui/button'
import { Alert, AlertDescription, AlertTitle } from '@/components/ui/alert'

interface ErrorProps {
  error: Error & { digest?: string }
  reset: () => void
}

export default function UsersError({ error, reset }: ErrorProps) {
  useEffect(() => {
    console.error(error)
  }, [error])

  return (
    <div className="flex flex-col items-center justify-center min-h-[400px] gap-4">
      <Alert variant="destructive" className="max-w-md">
        <AlertTitle>오류 발생</AlertTitle>
        <AlertDescription>{error.message || '알 수 없는 오류가 발생했습니다'}</AlertDescription>
      </Alert>
      <Button onClick={reset}>다시 시도</Button>
    </div>
  )
}
```

---

## components/UserTable.tsx (RSC 테이블)

```typescript
import Link from 'next/link'
import { Badge } from '@/components/ui/badge'
import { Button } from '@/components/ui/button'
import {
  Table,
  TableBody,
  TableCell,
  TableHead,
  TableHeader,
  TableRow,
} from '@/components/ui/table'
import type { User } from '@/types/user'

interface UserTableProps {
  users: User[]
  pagination?: { page: number; totalPages: number }
}

export function UserTable({ users, pagination }: UserTableProps) {
  return (
    <div className="rounded-md border">
      <Table>
        <TableHeader>
          <TableRow>
            <TableHead>이름</TableHead>
            <TableHead>이메일</TableHead>
            <TableHead>역할</TableHead>
            <TableHead>상태</TableHead>
            <TableHead className="text-right">작업</TableHead>
          </TableRow>
        </TableHeader>
        <TableBody>
          {users.length === 0 ? (
            <TableRow>
              <TableCell colSpan={5} className="text-center text-muted-foreground py-8">
                사용자가 없습니다
              </TableCell>
            </TableRow>
          ) : (
            users.map((user) => (
              <TableRow key={user.id}>
                <TableCell className="font-medium">{user.name}</TableCell>
                <TableCell>{user.email}</TableCell>
                <TableCell>
                  <Badge variant={user.role === 'ADMIN' ? 'default' : 'secondary'}>
                    {user.role}
                  </Badge>
                </TableCell>
                <TableCell>
                  <Badge variant={user.active ? 'outline' : 'destructive'}>
                    {user.active ? '활성' : '비활성'}
                  </Badge>
                </TableCell>
                <TableCell className="text-right">
                  <Button variant="ghost" size="sm" asChild>
                    <Link href={`/users/${user.id}`}>상세</Link>
                  </Button>
                </TableCell>
              </TableRow>
            ))
          )}
        </TableBody>
      </Table>
    </div>
  )
}
```

---

## components/UserDeleteButton.tsx (CC 삭제 버튼)

```typescript
'use client'

import { useState } from 'react'
import { useRouter } from 'next/navigation'
import { Button } from '@/components/ui/button'
import {
  AlertDialog,
  AlertDialogAction,
  AlertDialogCancel,
  AlertDialogContent,
  AlertDialogDescription,
  AlertDialogFooter,
  AlertDialogHeader,
  AlertDialogTitle,
  AlertDialogTrigger,
} from '@/components/ui/alert-dialog'
import { deleteUser } from '@/actions/user'
import { useToast } from '@/components/ui/use-toast'

interface UserDeleteButtonProps {
  userId: string
}

export function UserDeleteButton({ userId }: UserDeleteButtonProps) {
  const [isPending, setIsPending] = useState(false)
  const router = useRouter()
  const { toast } = useToast()

  async function handleDelete() {
    setIsPending(true)
    const result = await deleteUser(userId)
    setIsPending(false)

    if (result.error) {
      toast({ variant: 'destructive', title: '삭제 실패', description: result.error })
    } else {
      toast({ title: '삭제 완료', description: '사용자가 삭제되었습니다' })
      router.push('/users')
    }
  }

  return (
    <AlertDialog>
      <AlertDialogTrigger asChild>
        <Button variant="destructive" disabled={isPending}>삭제</Button>
      </AlertDialogTrigger>
      <AlertDialogContent>
        <AlertDialogHeader>
          <AlertDialogTitle>사용자 삭제</AlertDialogTitle>
          <AlertDialogDescription>
            이 사용자를 삭제하시겠습니까? 이 작업은 되돌릴 수 없습니다.
          </AlertDialogDescription>
        </AlertDialogHeader>
        <AlertDialogFooter>
          <AlertDialogCancel>취소</AlertDialogCancel>
          <AlertDialogAction onClick={handleDelete} disabled={isPending}>
            {isPending ? '삭제 중...' : '삭제'}
          </AlertDialogAction>
        </AlertDialogFooter>
      </AlertDialogContent>
    </AlertDialog>
  )
}
```

---

## app/(dashboard)/layout.tsx (Dashboard Layout)

```typescript
'use client'

import { useState } from 'react'
import Link from 'next/link'
import { usePathname } from 'next/navigation'
import { cn } from '@/lib/utils'
import { Button } from '@/components/ui/button'
import { Menu, X, Users, LayoutDashboard } from 'lucide-react'

const navItems = [
  { href: '/', label: '대시보드', icon: LayoutDashboard },
  { href: '/users', label: '사용자 관리', icon: Users },
]

export default function DashboardLayout({ children }: { children: React.ReactNode }) {
  const [sidebarOpen, setSidebarOpen] = useState(false)
  const pathname = usePathname()

  return (
    <div className="flex h-screen bg-background">
      {/* Sidebar */}
      <aside className={cn(
        'fixed inset-y-0 left-0 z-50 w-64 bg-card border-r transform transition-transform lg:static lg:translate-x-0',
        sidebarOpen ? 'translate-x-0' : '-translate-x-full',
      )}>
        <div className="flex h-16 items-center px-6 border-b">
          <span className="text-lg font-semibold">My App</span>
        </div>
        <nav className="p-4 space-y-1">
          {navItems.map((item) => (
            <Link
              key={item.href}
              href={item.href}
              className={cn(
                'flex items-center gap-3 px-3 py-2 rounded-md text-sm font-medium transition-colors',
                pathname === item.href
                  ? 'bg-primary text-primary-foreground'
                  : 'text-muted-foreground hover:bg-accent hover:text-accent-foreground',
              )}
            >
              <item.icon className="h-4 w-4" />
              {item.label}
            </Link>
          ))}
        </nav>
      </aside>

      {/* Main */}
      <div className="flex flex-1 flex-col overflow-hidden">
        <header className="flex h-16 items-center gap-4 border-b px-6">
          <Button
            variant="ghost"
            size="icon"
            className="lg:hidden"
            onClick={() => setSidebarOpen(!sidebarOpen)}
          >
            {sidebarOpen ? <X className="h-5 w-5" /> : <Menu className="h-5 w-5" />}
          </Button>
        </header>
        <main className="flex-1 overflow-y-auto p-6">{children}</main>
      </div>
    </div>
  )
}
```
