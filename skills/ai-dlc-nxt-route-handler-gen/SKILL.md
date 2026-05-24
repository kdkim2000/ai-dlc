---
name: ai-dlc-nxt-route-handler-gen
description: AI-DLC 개발단계(프론트엔드-Next.js) 스킬. Next.js Route Handlers(app/api/) 코드를 생성한다. "API 라우트 만들어줘", "Route Handler 생성", "Next.js API 엔드포인트", "app/api/ 코드 생성", "GET POST API 만들어줘" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Next.js Route Handlers 코드 생성

API 설계서(operationId·HTTP 메서드·경로)를 입력받아 `app/api/` 기반 Route Handler 코드를 생성한다.

## 트리거

- "API 라우트 만들어줘", "Route Handler 생성"
- "Next.js API 엔드포인트", "app/api/ 코드 생성"
- "GET POST API 만들어줘", "REST API Route 생성"

---

## 입력

### 필수
- API 설계서 (HTTP 메서드, 경로, 요청/응답 스펙)
- 도메인명 (예: users, products, orders)

### 선택
- 인증 필요 여부 (기본: Auth.js `auth()` 체크 포함)
- 외부 API 프록시 여부 (기본: 직접 DB 또는 외부 API 호출)

---

## 생성 절차

1. **파일 위치 결정**: `app/api/[domain]/route.ts` (목록), `app/api/[domain]/[id]/route.ts` (단건)
2. **HTTP 메서드 핸들러 작성**: `export async function GET/POST/PUT/DELETE(request: NextRequest)`
3. **인증 체크 삽입**: `auth()` 호출 → 미인증 시 401 반환
4. **요청 파싱**: URL params, searchParams, JSON body
5. **비즈니스 로직**: DB 조회 (Prisma) 또는 외부 API 프록시
6. **응답 반환**: `NextResponse.json(ApiResponse<T>)` 형식

---

## ApiResponse 타입

```typescript
// src/types/api.ts
export interface ApiResponse<T> {
  code: string       // 'SUCCESS' | 'ERROR' | 'VALIDATION_ERROR'
  message: string
  data: T | null
}

export function successResponse<T>(data: T, message = '성공'): ApiResponse<T> {
  return { code: 'SUCCESS', message, data }
}

export function errorResponse(message: string, code = 'ERROR'): ApiResponse<null> {
  return { code, message, data: null }
}
```

---

## 인증 체크 패턴

```typescript
import { auth } from '@/lib/auth'

export async function GET(request: NextRequest) {
  const session = await auth()
  if (!session) {
    return NextResponse.json(
      errorResponse('인증이 필요합니다', 'UNAUTHORIZED'),
      { status: 401 }
    )
  }
  // ...
}
```

---

## 에러 처리 패턴

```typescript
try {
  // 비즈니스 로직
} catch (error) {
  if (error instanceof Prisma.PrismaClientKnownRequestError) {
    if (error.code === 'P2025') {
      return NextResponse.json(errorResponse('리소스를 찾을 수 없습니다'), { status: 404 })
    }
  }
  console.error('[API Error]', error)
  return NextResponse.json(errorResponse('서버 오류가 발생했습니다'), { status: 500 })
}
```

---

## Route Handler vs Server Action 선택 기준

| 상황 | 권장 |
|:---|:---|
| 브라우저 폼 제출, 내부 뮤테이션 | Server Actions |
| 외부 클라이언트(모바일·타 서비스) API 노출 | Route Handlers |
| 파일 업로드 (multipart/form-data) | Route Handlers |
| 웹훅 수신 (Stripe, GitHub 등) | Route Handlers |
| CORS가 필요한 public API | Route Handlers |

---

## 생성 파일 목록

| 파일 | 메서드 | 설명 |
|:---|:---|:---|
| `app/api/[domain]/route.ts` | GET, POST | 목록 조회, 생성 |
| `app/api/[domain]/[id]/route.ts` | GET, PUT, DELETE | 단건 조회·수정·삭제 |
| `src/types/api.ts` | - | ApiResponse 타입 + 헬퍼 함수 |

---

## 산출물

- `app/api/**/*.ts` — Route Handler 파일
- `src/types/api.ts` — 공통 API 응답 타입
