# Next.js App Router Playwright e2e 테스트 템플릿

---

## playwright.config.ts

```typescript
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './tests',
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
    timeout: 120 * 1000,
  },
})
```

---

## tests/fixtures/auth.ts (인증 픽스처)

```typescript
import { test as base, type Page } from '@playwright/test'

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

export { expect } from '@playwright/test'
```

---

## tests/pages/BasePage.ts

```typescript
import { type Page, type Locator } from '@playwright/test'

export class BasePage {
  readonly page: Page

  constructor(page: Page) {
    this.page = page
  }

  async navigate(path: string) {
    await this.page.goto(path)
    await this.page.waitForLoadState('networkidle')
  }

  getByTestId(testId: string): Locator {
    return this.page.getByTestId(testId)
  }

  async waitForToast(text: string) {
    await this.page.getByRole('status').filter({ hasText: text }).waitFor({ timeout: 5000 })
  }
}
```

---

## tests/pages/UsersPage.ts

```typescript
import { type Page, type Locator } from '@playwright/test'
import { BasePage } from './BasePage'

export class UsersPage extends BasePage {
  readonly table: Locator
  readonly addButton: Locator
  readonly searchInput: Locator

  constructor(page: Page) {
    super(page)
    this.table = page.getByTestId('user-table')
    this.addButton = page.getByRole('link', { name: '사용자 등록' })
    this.searchInput = page.getByTestId('search-input')
  }

  async goto() {
    await this.navigate('/users')
  }

  async clickAdd() {
    await this.addButton.click()
    await this.page.waitForURL('/users/new')
  }

  async search(keyword: string) {
    await this.searchInput.fill(keyword)
    await this.page.waitForTimeout(300)  // debounce
  }

  getRow(name: string): Locator {
    return this.table.getByRole('row').filter({ hasText: name })
  }
}

export class CreateUserPage extends BasePage {
  readonly nameInput: Locator
  readonly emailInput: Locator
  readonly roleSelect: Locator
  readonly submitButton: Locator

  constructor(page: Page) {
    super(page)
    this.nameInput = page.getByLabel('이름')
    this.emailInput = page.getByLabel('이메일')
    this.roleSelect = page.getByLabel('역할')
    this.submitButton = page.getByRole('button', { name: '사용자 등록' })
  }

  async goto() {
    await this.navigate('/users/new')
  }

  async fillForm(data: { name: string; email: string; role?: string }) {
    await this.nameInput.fill(data.name)
    await this.emailInput.fill(data.email)
    if (data.role) {
      await this.roleSelect.selectOption(data.role)
    }
  }

  async submit() {
    await this.submitButton.click()
  }
}
```

---

## tests/users.spec.ts

```typescript
import { expect } from '@playwright/test'
import { test } from './fixtures/auth'
import { UsersPage, CreateUserPage } from './pages/UsersPage'

test.describe('사용자 관리', () => {
  test('목록 페이지 진입 — 테이블 표시', async ({ authenticatedPage: page }) => {
    const usersPage = new UsersPage(page)
    await usersPage.goto()

    await expect(usersPage.table).toBeVisible()
    await expect(page.getByRole('heading', { name: '사용자 관리' })).toBeVisible()
    await expect(usersPage.addButton).toBeVisible()
  })

  test('사용자 등록 — Happy Path', async ({ authenticatedPage: page }) => {
    const createPage = new CreateUserPage(page)
    await createPage.goto()

    await createPage.fillForm({
      name: '홍길동',
      email: `test-${Date.now()}@example.com`,
      role: 'USER',
    })
    await createPage.submit()

    // Server Action 성공 → redirect
    await page.waitForURL('/users')

    const usersPage = new UsersPage(page)
    await expect(usersPage.getRow('홍길동')).toBeVisible()
  })

  test('사용자 등록 — 유효성 검사 오류', async ({ authenticatedPage: page }) => {
    const createPage = new CreateUserPage(page)
    await createPage.goto()

    // 빈 폼 제출
    await createPage.submit()

    // 필드 오류 메시지 표시
    await expect(page.getByText('이름은 2자 이상이어야 합니다')).toBeVisible()
    await expect(page.getByText('유효한 이메일 형식을 입력하세요')).toBeVisible()

    // 페이지 이동 없음
    await expect(page).toHaveURL('/users/new')
  })

  test('사용자 삭제 — 확인 Dialog 후 삭제', async ({ authenticatedPage: page }) => {
    // 목록에서 첫 번째 사용자 상세로 이동
    const usersPage = new UsersPage(page)
    await usersPage.goto()

    await page.getByRole('link', { name: '상세' }).first().click()
    await page.waitForURL(/\/users\/.+/)

    // 삭제 버튼 클릭 → Dialog 열림
    await page.getByRole('button', { name: '삭제' }).click()
    await expect(page.getByRole('alertdialog')).toBeVisible()

    // 확인 클릭 → 삭제 후 목록으로 이동
    await page.getByRole('button', { name: '삭제', exact: true }).last().click()
    await page.waitForURL('/users')
  })

  test('미인증 — 로그인 페이지로 리다이렉트', async ({ page }) => {
    await page.goto('/users')
    await page.waitForURL('/login')
    await expect(page).toHaveURL('/login')
  })
})
```
