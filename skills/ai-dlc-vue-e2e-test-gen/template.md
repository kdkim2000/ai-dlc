# Vue Playwright e2e 테스트 코드 템플릿

## playwright.config.ts

```typescript
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './tests/specs',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: 'html',
  use: {
    baseURL: 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120_000,
  },
})
```

---

## tests/fixtures/auth.ts

```typescript
import { test as base, type Page } from '@playwright/test'

type AuthFixtures = {
  authenticatedPage: Page
}

export const test = base.extend<AuthFixtures>({
  authenticatedPage: async ({ page }, use) => {
    // 로그인 페이지에서 인증
    await page.goto('/login')
    await page.getByTestId('input-email').fill('admin@example.com')
    await page.getByTestId('input-password').fill('password123')
    await page.getByTestId('btn-login').click()
    await page.waitForURL('/')
    await use(page)
  },
})

export { expect } from '@playwright/test'
```

---

## tests/pages/BasePage.ts

```typescript
import type { Page, Locator } from '@playwright/test'
import { expect } from '@playwright/test'

export class BasePage {
  protected readonly page: Page

  constructor(page: Page) {
    this.page = page
  }

  async navigate(path: string) {
    await this.page.goto(path)
    await this.page.waitForLoadState('networkidle')
  }

  async waitForToast(message: string) {
    const toast = this.page.getByRole('status').filter({ hasText: message })
    await expect(toast).toBeVisible({ timeout: 5000 })
  }

  async clickButton(testId: string) {
    await this.page.getByTestId(testId).click()
  }

  async fillInput(testId: string, value: string) {
    await this.page.getByTestId(testId).fill(value)
  }

  getByTestId(testId: string): Locator {
    return this.page.getByTestId(testId)
  }
}
```

---

## tests/pages/UsersPage.ts

```typescript
import type { Page } from '@playwright/test'
import { expect } from '@playwright/test'
import { BasePage } from './BasePage'

export class UsersPage extends BasePage {
  constructor(page: Page) {
    super(page)
  }

  async goto() {
    await this.navigate('/users')
  }

  async waitForTableLoad() {
    await expect(this.page.getByTestId('user-table')).toBeVisible()
  }

  async clickCreateButton() {
    await this.page.getByTestId('btn-create').click()
    await this.page.waitForURL('/users/new')
  }

  async clickDeleteButton(userId: number) {
    await this.page.getByTestId(`btn-delete-${userId}`).click()
  }

  async confirmDelete() {
    await this.page.getByTestId('btn-confirm').click()
  }

  async getUserRowCount() {
    return await this.page.locator('[data-testid="user-table"] tbody tr').count()
  }
}

export class CreateUserPage extends BasePage {
  constructor(page: Page) {
    super(page)
  }

  async goto() {
    await this.navigate('/users/new')
  }

  async fillForm(data: { name: string; email: string; role: string }) {
    await this.fillInput('input-name', data.name)
    await this.fillInput('input-email', data.email)
    await this.page.getByTestId('select-role').click()
    await this.page.getByRole('option', { name: data.role }).click()
  }

  async submit() {
    await this.clickButton('btn-submit')
  }

  async getValidationError(field: string): Promise<string | null> {
    const error = this.page.locator(`[data-testid="input-${field}"] ~ [role="alert"]`)
    const isVisible = await error.isVisible()
    return isVisible ? await error.textContent() : null
  }
}
```

---

## tests/specs/users.spec.ts

```typescript
import { expect } from '@playwright/test'
import { test } from '../fixtures/auth'
import { UsersPage, CreateUserPage } from '../pages/UsersPage'

test.describe('사용자 관리', () => {
  test('목록 페이지 정상 로드', async ({ authenticatedPage }) => {
    const usersPage = new UsersPage(authenticatedPage)
    await usersPage.goto()
    await usersPage.waitForTableLoad()
    expect(await usersPage.getUserRowCount()).toBeGreaterThanOrEqual(0)
  })

  test('사용자 등록 성공 - Happy Path', async ({ authenticatedPage }) => {
    const usersPage = new UsersPage(authenticatedPage)
    const createPage = new CreateUserPage(authenticatedPage)

    await usersPage.goto()
    await usersPage.clickCreateButton()

    await createPage.fillForm({
      name: '홍길동',
      email: `user_${Date.now()}@example.com`,
      role: '일반 사용자',
    })
    await createPage.submit()

    await authenticatedPage.waitForURL('/users')
    await usersPage.waitForTableLoad()
  })

  test('사용자 등록 실패 - 유효성 오류', async ({ authenticatedPage }) => {
    const createPage = new CreateUserPage(authenticatedPage)
    await createPage.goto()

    // 빈 폼으로 제출
    await createPage.submit()

    const nameError = await createPage.getValidationError('name')
    expect(nameError).toContain('이름을 입력하세요')

    const emailError = await createPage.getValidationError('email')
    expect(emailError).toContain('이메일')
  })

  test('사용자 등록 실패 - 이메일 형식 오류', async ({ authenticatedPage }) => {
    const createPage = new CreateUserPage(authenticatedPage)
    await createPage.goto()

    await createPage.fillInput('input-email', 'invalid-email')
    await createPage.submit()

    const emailError = await createPage.getValidationError('email')
    expect(emailError).toContain('올바른 이메일 형식')
  })

  test('비인증 접근 시 로그인 페이지로 리다이렉트', async ({ page }) => {
    // authenticatedPage fixture 없이 일반 page 사용 (비인증 상태)
    await page.goto('/users')
    await page.waitForURL('/login**')
    expect(page.url()).toContain('/login')
  })
})

test.describe('사용자 삭제', () => {
  test('삭제 확인 다이얼로그 표시 후 삭제', async ({ authenticatedPage }) => {
    const usersPage = new UsersPage(authenticatedPage)
    await usersPage.goto()
    await usersPage.waitForTableLoad()

    const initialCount = await usersPage.getUserRowCount()
    if (initialCount === 0) {
      test.skip()
      return
    }

    // 첫 번째 행의 삭제 버튼 클릭 (id는 실제 데이터에 따라 다름)
    await authenticatedPage.locator('[data-testid^="btn-delete-"]').first().click()

    // 확인 다이얼로그 표시 확인
    await expect(authenticatedPage.getByRole('dialog')).toBeVisible()
    await expect(authenticatedPage.getByText('삭제한 사용자는 복구할 수 없습니다')).toBeVisible()

    // 확인 버튼 클릭
    await usersPage.confirmDelete()

    // 목록 새로고침 후 건수 감소 확인
    await usersPage.waitForTableLoad()
    const finalCount = await usersPage.getUserRowCount()
    expect(finalCount).toBe(initialCount - 1)
  })
})
```

---

## tests/specs/auth.spec.ts

```typescript
import { test, expect } from '@playwright/test'

test.describe('인증', () => {
  test('로그인 성공', async ({ page }) => {
    await page.goto('/login')
    await page.getByTestId('input-email').fill('admin@example.com')
    await page.getByTestId('input-password').fill('password123')
    await page.getByTestId('btn-login').click()
    await page.waitForURL('/')
    await expect(page).toHaveURL('/')
  })

  test('로그인 실패 - 잘못된 비밀번호', async ({ page }) => {
    await page.goto('/login')
    await page.getByTestId('input-email').fill('admin@example.com')
    await page.getByTestId('input-password').fill('wrongpassword')
    await page.getByTestId('btn-login').click()
    await expect(page.getByRole('alert')).toBeVisible()
    await expect(page).toHaveURL('/login')
  })

  test('로그인 페이지 유효성 오류', async ({ page }) => {
    await page.goto('/login')
    await page.getByTestId('btn-login').click()
    await expect(page.getByText('이메일을 입력하세요')).toBeVisible()
  })
})
```
