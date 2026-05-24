# Next.js Route Handlers 코드 템플릿

---

## src/types/api.ts (공통 응답 타입)

```typescript
export interface ApiResponse<T> {
  code: string
  message: string
  data: T | null
}

export interface PaginatedData<T> {
  items: T[]
  pagination: {
    page: number
    pageSize: number
    totalCount: number
    totalPages: number
  }
}

export function successResponse<T>(data: T, message = '성공'): ApiResponse<T> {
  return { code: 'SUCCESS', message, data }
}

export function errorResponse(message: string, code = 'ERROR'): ApiResponse<null> {
  return { code, message, data: null }
}
```

---

## app/api/users/route.ts (목록 GET + 생성 POST)

```typescript
import { NextRequest, NextResponse } from 'next/server'
import { auth } from '@/lib/auth'
import { prisma } from '@/lib/prisma'
import { z } from 'zod'
import { successResponse, errorResponse } from '@/types/api'

const CreateUserSchema = z.object({
  name: z.string().min(2),
  email: z.string().email(),
  role: z.enum(['ADMIN', 'USER']).default('USER'),
})

export async function GET(request: NextRequest) {
  const session = await auth()
  if (!session) {
    return NextResponse.json(errorResponse('인증이 필요합니다', 'UNAUTHORIZED'), { status: 401 })
  }

  const { searchParams } = request.nextUrl
  const page = Number(searchParams.get('page') ?? 1)
  const pageSize = Number(searchParams.get('pageSize') ?? 20)
  const keyword = searchParams.get('keyword') ?? undefined

  try {
    const where = keyword
      ? { OR: [{ name: { contains: keyword } }, { email: { contains: keyword } }] }
      : {}

    const [items, totalCount] = await Promise.all([
      prisma.user.findMany({
        where,
        skip: (page - 1) * pageSize,
        take: pageSize,
        orderBy: { createdAt: 'desc' },
      }),
      prisma.user.count({ where }),
    ])

    return NextResponse.json(
      successResponse({
        items,
        pagination: { page, pageSize, totalCount, totalPages: Math.ceil(totalCount / pageSize) },
      }),
    )
  } catch (error) {
    console.error('[GET /api/users]', error)
    return NextResponse.json(errorResponse('서버 오류가 발생했습니다'), { status: 500 })
  }
}

export async function POST(request: NextRequest) {
  const session = await auth()
  if (!session) {
    return NextResponse.json(errorResponse('인증이 필요합니다', 'UNAUTHORIZED'), { status: 401 })
  }

  let body: unknown
  try {
    body = await request.json()
  } catch {
    return NextResponse.json(errorResponse('잘못된 요청 형식입니다'), { status: 400 })
  }

  const parsed = CreateUserSchema.safeParse(body)
  if (!parsed.success) {
    return NextResponse.json(
      errorResponse('입력값이 유효하지 않습니다', 'VALIDATION_ERROR'),
      { status: 422 },
    )
  }

  try {
    const user = await prisma.user.create({ data: parsed.data })
    return NextResponse.json(successResponse(user, '사용자가 등록되었습니다'), { status: 201 })
  } catch (error: unknown) {
    if (
      typeof error === 'object' &&
      error !== null &&
      'code' in error &&
      (error as { code: string }).code === 'P2002'
    ) {
      return NextResponse.json(errorResponse('이미 존재하는 이메일입니다'), { status: 409 })
    }
    console.error('[POST /api/users]', error)
    return NextResponse.json(errorResponse('서버 오류가 발생했습니다'), { status: 500 })
  }
}
```

---

## app/api/users/[id]/route.ts (단건 GET + PUT + DELETE)

```typescript
import { NextRequest, NextResponse } from 'next/server'
import { auth } from '@/lib/auth'
import { prisma } from '@/lib/prisma'
import { z } from 'zod'
import { successResponse, errorResponse } from '@/types/api'

const UpdateUserSchema = z.object({
  name: z.string().min(2).optional(),
  role: z.enum(['ADMIN', 'USER']).optional(),
  active: z.boolean().optional(),
})

interface RouteContext {
  params: { id: string }
}

export async function GET(_request: NextRequest, { params }: RouteContext) {
  const session = await auth()
  if (!session) {
    return NextResponse.json(errorResponse('인증이 필요합니다', 'UNAUTHORIZED'), { status: 401 })
  }

  try {
    const user = await prisma.user.findUnique({ where: { id: params.id } })
    if (!user) {
      return NextResponse.json(errorResponse('사용자를 찾을 수 없습니다'), { status: 404 })
    }
    return NextResponse.json(successResponse(user))
  } catch (error) {
    console.error('[GET /api/users/:id]', error)
    return NextResponse.json(errorResponse('서버 오류가 발생했습니다'), { status: 500 })
  }
}

export async function PUT(request: NextRequest, { params }: RouteContext) {
  const session = await auth()
  if (!session) {
    return NextResponse.json(errorResponse('인증이 필요합니다', 'UNAUTHORIZED'), { status: 401 })
  }

  let body: unknown
  try {
    body = await request.json()
  } catch {
    return NextResponse.json(errorResponse('잘못된 요청 형식입니다'), { status: 400 })
  }

  const parsed = UpdateUserSchema.safeParse(body)
  if (!parsed.success) {
    return NextResponse.json(
      errorResponse('입력값이 유효하지 않습니다', 'VALIDATION_ERROR'),
      { status: 422 },
    )
  }

  try {
    const user = await prisma.user.update({
      where: { id: params.id },
      data: parsed.data,
    })
    return NextResponse.json(successResponse(user, '사용자 정보가 수정되었습니다'))
  } catch (error: unknown) {
    if (
      typeof error === 'object' &&
      error !== null &&
      'code' in error &&
      (error as { code: string }).code === 'P2025'
    ) {
      return NextResponse.json(errorResponse('사용자를 찾을 수 없습니다'), { status: 404 })
    }
    console.error('[PUT /api/users/:id]', error)
    return NextResponse.json(errorResponse('서버 오류가 발생했습니다'), { status: 500 })
  }
}

export async function DELETE(_request: NextRequest, { params }: RouteContext) {
  const session = await auth()
  if (!session) {
    return NextResponse.json(errorResponse('인증이 필요합니다', 'UNAUTHORIZED'), { status: 401 })
  }

  try {
    await prisma.user.delete({ where: { id: params.id } })
    return NextResponse.json(successResponse(null, '사용자가 삭제되었습니다'))
  } catch (error: unknown) {
    if (
      typeof error === 'object' &&
      error !== null &&
      'code' in error &&
      (error as { code: string }).code === 'P2025'
    ) {
      return NextResponse.json(errorResponse('사용자를 찾을 수 없습니다'), { status: 404 })
    }
    console.error('[DELETE /api/users/:id]', error)
    return NextResponse.json(errorResponse('서버 오류가 발생했습니다'), { status: 500 })
  }
}
```

---

## src/lib/prisma.ts (Prisma 클라이언트 싱글턴)

```typescript
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as { prisma: PrismaClient }

export const prisma = globalForPrisma.prisma ?? new PrismaClient()

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```
