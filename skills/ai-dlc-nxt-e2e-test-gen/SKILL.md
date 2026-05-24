---
name: ai-dlc-nxt-e2e-test-gen
description: AI-DLC 개발단계(프론트엔드-Next.js) 스킬. Playwright e2e 테스트 코드를 생성한다(Next.js App Router용). "Next.js e2e 테스트 만들어줘", "App Router Playwright 테스트", "Next.js 통합 테스트 생성", "Playwright 테스트 코드 작성", "e2e 테스트 생성" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Next.js App Router Playwright e2e 테스트 생성

화면설계서(SCR-NNN)·유즈케이스(UC-NNN) 기반으로 Next.js App Router에 최적화된 Playwright e2e 테스트 코드를 생성한다.

## 트리거

- "Next.js e2e 테스트 만들어줘", "App Router Playwright 테스트"
- "Next.js 통합 테스트 생성", "Playwright 테스트 코드 작성"

---

## 입력

### 필수
- 화면설계서 (SCR-NNN) 또는 테스트할 기능 설명
- 테스트 대상 URL 경로

### 선택
- 유즈케이스 (UC-NNN)
- Server Action 또는 Route Handler 목록

---

## 생성 절차

1. **`playwright.config.ts` 확인/생성**: `webServer`를 `next dev --turbopack`으로 설정
2. **Page Object 클래스 작성**: `tests/pages/[Domain]Page.ts`
   - `data-testid` 선택자 사용 (CSS 클래스 선택자 금지)
   - 각 화면별 메서드 (navigate, fill, submit, assert)
3. **테스트 시나리오 작성**: `tests/[domain].spec.ts`
   - Happy Path (정상 흐름) 필수
   - 유효성 검사 오류 시나리오
   - 인증 리다이렉트 테스트
   - Server Action 폼 제출 + 결과 검증

---

## Next.js 특화 테스트 패턴

| 상황 | 테스트 방법 |
|:---|:---|
| 로그인 인증 필요 페이지 | `storageState` 또는 `test.beforeEach` 로그인 |
| Server Action 폼 제출 | `page.fill` + `page.getByRole('button').click()` + URL 변경 대기 |
| 서버 리다이렉트 확인 | `page.waitForURL('/target')` |
| RSC 데이터 표시 확인 | `expect(page.getByTestId('user-table')).toBeVisible()` |
| 로딩 스켈레톤 확인 | `expect(page.getByTestId('skeleton')).toBeVisible()` |
| Toast 알림 확인 | `expect(page.getByRole('status')).toContainText('성공')` |

---

## 인증 상태 설정 패턴

```typescript
// tests/fixtures/auth.ts
import { test as base, Page } from '@playwright/test'

export const test = base.extend<{ authenticatedPage: Page }>({
  authenticatedPage: async ({ page }, use) => {
    await page.goto('/login')
    await page.getByLabel('아이디').fill('admin')
    await page.getByLabel('비밀번호').fill('password')
    await page.getByRole('button', { name: '로그인' }).click()
    await page.waitForURL('/')
    await use(page)
  },
})
```

---

## 코드 원칙

- `data-testid` 속성으로만 요소 선택 (CSS 클래스·XPath 금지)
- 각 테스트는 독립적 (beforeEach에서 상태 초기화)
- Server Action 완료는 URL 변경 또는 Toast로 확인
- `page.waitForURL()` 사용 (sleep 금지)
- 실패 시 스크린샷 자동 캡처 (`screenshot: 'only-on-failure'`)

---

## 생성 파일 목록

| 파일 | 설명 |
|:---|:---|
| `playwright.config.ts` | webServer: next dev, baseURL 설정 |
| `tests/fixtures/auth.ts` | 인증된 페이지 픽스처 |
| `tests/pages/BasePage.ts` | 공통 Page Object (navigate, waitForLoad) |
| `tests/pages/[Domain]Page.ts` | 도메인 Page Object |
| `tests/[domain].spec.ts` | e2e 테스트 시나리오 |

---

## 산출물

- `playwright.config.ts`
- `tests/**/*.ts` — Page Object + 테스트 파일
