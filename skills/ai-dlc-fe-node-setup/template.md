# Node.js/Express 서버 설정 템플릿

## package.json

```json
{
  "name": "{{프로젝트명}}-server",
  "version": "0.1.0",
  "private": true,
  "scripts": {
    "dev": "ts-node-dev --respawn --transpile-only src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js"
  },
  "dependencies": {
    "express": "^4.21.0",
    "cors": "^2.8.5",
    "morgan": "^1.10.0",
    "dotenv": "^16.4.5",
    "helmet": "^8.0.0",
    "http-proxy-middleware": "^3.0.3"
  },
  "devDependencies": {
    "@types/express": "^5.0.0",
    "@types/cors": "^2.8.17",
    "@types/morgan": "^1.9.9",
    "@types/node": "^22.7.5",
    "typescript": "^5.6.3",
    "ts-node-dev": "^2.0.0"
  }
}
```

## tsconfig.json (Node용)

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "dist",
    "rootDir": "src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "resolveJsonModule": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## src/types/api-response.ts

```typescript
export interface ApiResponse<T = unknown> {
  code: string
  message: string
  data: T
}

export function ok<T>(data: T, message = 'success'): ApiResponse<T> {
  return { code: '200', message, data }
}

export function created<T>(data: T, message = 'created'): ApiResponse<T> {
  return { code: '201', message, data }
}

export function error(message: string, code = '500'): ApiResponse<null> {
  return { code, message, data: null }
}
```

## src/app.ts

```typescript
import express from 'express'
import cors from 'cors'
import morgan from 'morgan'
import helmet from 'helmet'
import routes from './routes'
import { errorHandler } from './middleware/errorHandler'

const app = express()

// 보안 헤더
app.use(helmet())

// CORS: React 개발 서버 허용
app.use(cors({
  origin: process.env.ALLOWED_ORIGIN ?? 'http://localhost:3000',
  credentials: true,
}))

// 요청 로깅
app.use(morgan('dev'))

// JSON 파싱
app.use(express.json())
app.use(express.urlencoded({ extended: true }))

// 라우터
app.use('/api', routes)

// 헬스체크
app.get('/health', (_req, res) => {
  res.json({ status: 'ok', timestamp: new Date().toISOString() })
})

// 에러 핸들러 (마지막에 등록)
app.use(errorHandler)

export default app
```

## src/server.ts

```typescript
import 'dotenv/config'
import app from './app'

const PORT = Number(process.env.PORT ?? 4000)

app.listen(PORT, () => {
  console.log(`Server running on http://localhost:${PORT}`)
})
```

## src/routes/index.ts

```typescript
import { Router } from 'express'
// 도메인별 라우터 import 예시:
// import userRouter from './user.router'

const router = Router()

// router.use('/users', userRouter)

// Mock 라우트 예시
router.get('/ping', (_req, res) => {
  res.json({ message: 'pong' })
})

export default router
```

## src/routes/user.router.ts (Mock 라우터 예시)

```typescript
import { Router } from 'express'
import { ok, created } from '../types/api-response'

const router = Router()

const mockUsers = [
  { userId: 1, userNm: '홍길동', email: 'hong@example.com' },
  { userId: 2, userNm: '김철수', email: 'kim@example.com' },
]

// 목록 조회
router.get('/', (_req, res) => {
  res.json(ok(mockUsers))
})

// 단건 조회
router.get('/:id', (req, res) => {
  const user = mockUsers.find((u) => u.userId === Number(req.params.id))
  if (!user) {
    res.status(404).json({ code: '404', message: '사용자를 찾을 수 없습니다.', data: null })
    return
  }
  res.json(ok(user))
})

// 등록
router.post('/', (req, res) => {
  const newUser = { userId: mockUsers.length + 1, ...req.body }
  mockUsers.push(newUser)
  res.status(201).json(created(newUser))
})

// 수정
router.put('/:id', (req, res) => {
  const idx = mockUsers.findIndex((u) => u.userId === Number(req.params.id))
  if (idx === -1) {
    res.status(404).json({ code: '404', message: '사용자를 찾을 수 없습니다.', data: null })
    return
  }
  mockUsers[idx] = { ...mockUsers[idx], ...req.body }
  res.json(ok(mockUsers[idx]))
})

// 삭제
router.delete('/:id', (req, res) => {
  const idx = mockUsers.findIndex((u) => u.userId === Number(req.params.id))
  if (idx !== -1) mockUsers.splice(idx, 1)
  res.status(204).send()
})

export default router
```

## src/middleware/errorHandler.ts

```typescript
import { Request, Response, NextFunction } from 'express'

interface AppError extends Error {
  status?: number
  code?: string
}

export function errorHandler(
  err: AppError,
  _req: Request,
  res: Response,
  _next: NextFunction,
): void {
  const status = err.status ?? 500
  const code = err.code ?? String(status)
  const message = err.message ?? '서버 오류가 발생했습니다.'

  console.error(`[Error] ${status} - ${message}`, err.stack)

  res.status(status).json({ code, message, data: null })
}
```

## .env.example

```
PORT=4000
ALLOWED_ORIGIN=http://localhost:3000
# BFF 용도일 때 프록시 타겟
TARGET_API_URL=http://localhost:8080
```

## BFF 프록시 설정 (src/routes/proxy.router.ts) — BFF 목적 시 참조

```typescript
import { Router } from 'express'
import { createProxyMiddleware } from 'http-proxy-middleware'

const router = Router()

router.use(
  createProxyMiddleware({
    target: process.env.TARGET_API_URL ?? 'http://localhost:8080',
    changeOrigin: true,
    pathRewrite: { '^/api': '' },
    on: {
      error: (err, _req, res) => {
        (res as Response).status(502).json({
          code: '502',
          message: '백엔드 서버 연결 오류',
          data: null,
        })
      },
    },
  }),
)

export default router
```
