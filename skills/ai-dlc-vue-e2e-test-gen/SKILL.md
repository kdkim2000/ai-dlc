---
name: ai-dlc-vue-e2e-test-gen
description: AI-DLC 개발단계(프론트엔드-Vue.js) 스킬. Vue.js 앱을 위한 Playwright e2e 테스트 코드를 생성한다. "Vue e2e 테스트 만들어줘", "Vue Playwright 테스트", "Vue 통합 테스트 생성", "뷰 e2e 테스트", "Vue 자동화 테스트 만들어줘", "Playwright Vue 테스트 코드" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Vue.js Playwright e2e 테스트 생성

화면설계서(SCR-NNN)·Vue 구현 코드를 분석하여 Page Object Model 기반 Playwright e2e 테스트를 생성한다.
공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "Vue e2e 테스트 만들어줘", "Vue Playwright 테스트", "Vue 통합 테스트 생성"
- "뷰 e2e 테스트", "Vue 자동화 테스트 만들어줘"
- "Playwright Vue 테스트 코드", "Vue 화면 테스트 자동화"

---

## 입력

### 필수
- 테스트 대상 화면 또는 기능 (화면ID SCR-NNN 또는 기능 설명)
- Vue 구현 코드 (View 컴포넌트, 라우트 경로)

### 선택
- API 명세 (Mock 서버 설정 시 활용)
- 화면설계서 (테스트 시나리오 도출)

---

## 분석 절차

### 1단계: 테스트 대상 파악
- 테스트할 화면·기능 목록
- Happy Path (정상 흐름) 시나리오
- 유효성 오류 시나리오
- 인증 관련 시나리오 (비인증 리다이렉트 등)

### 2단계: playwright.config.ts 설정
```typescript
// Vue 개발 서버 (vite dev) 자동 시작
webServer: {
  command: 'npm run dev',
  url: 'http://localhost:3000',
  reuseExistingServer: !process.env.CI,
  timeout: 120_000,
}
```

### 3단계: Page Object Model (POM) 구조
```
tests/
├── playwright.config.ts
├── fixtures/
│   └── auth.ts          # 인증된 페이지 fixture
├── pages/
│   ├── BasePage.ts       # 공통 메서드 (navigate, waitForLoad 등)
│   ├── LoginPage.ts      # 로그인 페이지 메서드
│   └── UsersPage.ts      # 사용자 관련 페이지 메서드
└── specs/
    ├── auth.spec.ts
    └── users.spec.ts
```

### 4단계: Vue Router URL 구조 반영
- Vue Router `createWebHistory` 기반 URL 사용
- 라우트 경로: `/users`, `/users/new`, `/users/:id` 등
- `page.waitForURL('/users')` 또는 정규식 패턴 사용

### 5단계: Vue SFC 특성 고려
- 데이터 패칭 완료 대기: `page.waitForSelector('[data-testid="user-table"]')`
- Vue Router 네비게이션 완료 대기: `page.waitForURL()`
- `v-if` 조건부 렌더링: 요소 가시성 확인 후 조작
- 폼 submit (VeeValidate): 비동기 검증 후 submit 버튼 활성화 대기

### 6단계: 테스트 시나리오 작성
각 테스트 파일 구조:
```typescript
import { test, expect } from '@playwright/test'
import { UsersPage } from './pages/UsersPage'

test.describe('사용자 관리', () => {
  test('목록 조회', async ({ page }) => { ... })
  test('등록 성공', async ({ page }) => { ... })
  test('등록 실패 - 유효성 오류', async ({ page }) => { ... })
  test('삭제 확인', async ({ page }) => { ... })
  test('비인증 접근 시 로그인 페이지 리다이렉트', async ({ page }) => { ... })
})
```

---

## 생성 원칙

| 원칙 | 내용 |
|:---|:---|
| Page Object Model | 셀렉터·액션을 Page 클래스로 캡슐화 |
| data-testid 속성 | SFC template에 `data-testid` 추가 권장 |
| 명확한 대기 | `waitForURL`, `waitForSelector`, `expect(locator).toBeVisible()` |
| 독립적 테스트 | 각 test는 독립 실행 가능, 순서 의존 금지 |
| 인증 fixture | `authenticatedPage` fixture로 로그인 상태 재사용 |
| API Mock | `page.route()` 또는 실제 API 서버 선택 (환경별) |

---

## 산출물

- `playwright.config.ts`
- `tests/fixtures/auth.ts`
- `tests/pages/BasePage.ts`
- `tests/pages/{Domain}Page.ts`
- `tests/specs/{domain}.spec.ts`

template.md에서 코드 골격을 참조한다.
