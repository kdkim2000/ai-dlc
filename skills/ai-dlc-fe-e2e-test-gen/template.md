# Playwright e2e 테스트 템플릿

## playwright.config.ts

```typescript
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './tests',
  fullyParallel: true,
  forbidOnly: !!process.env.CI,
  retries: process.env.CI ? 2 : 0,
  workers: process.env.CI ? 1 : undefined,
  reporter: process.env.CI
    ? [['github'], ['html', { open: 'never' }]]
    : [['html', { open: 'on-failure' }]],
  use: {
    baseURL: process.env.BASE_URL ?? 'http://localhost:3000',
    trace: 'on-first-retry',
    screenshot: 'only-on-failure',
    video: 'retain-on-failure',
    locale: 'ko-KR',
  },
  projects: [
    { name: 'chromium', use: { ...devices['Desktop Chrome'] } },
    // { name: 'firefox', use: { ...devices['Desktop Firefox'] } },
    // { name: 'mobile', use: { ...devices['iPhone 14'] } },
  ],
  webServer: {
    command: 'npm run dev',
    url: 'http://localhost:3000',
    reuseExistingServer: !process.env.CI,
    timeout: 120 * 1000,
  },
})
```

## Page Object 기본 클래스 (`tests/pages/BasePage.ts`)

```typescript
import { type Page } from '@playwright/test'

export abstract class BasePage {
  constructor(protected page: Page) {}

  async waitForPageLoad() {
    await this.page.waitForLoadState('networkidle')
  }

  async getToastMessage() {
    const toast = this.page.locator('[data-radix-toast-viewport] [role="status"]').first()
    await toast.waitFor({ state: 'visible', timeout: 5000 })
    return toast.textContent()
  }
}
```

## Page Object 예시 (`tests/pages/UserListPage.ts`)

```typescript
import { type Page, type Locator, expect } from '@playwright/test'
import { BasePage } from './BasePage'

export class UserListPage extends BasePage {
  readonly btnCreate: Locator
  readonly searchInput: Locator
  readonly tableRows: Locator

  constructor(page: Page) {
    super(page)
    this.btnCreate = page.getByTestId('btn-create')
    this.searchInput = page.getByTestId('input-search')
    this.tableRows = page.locator('tbody tr')
  }

  async goto() {
    await this.page.goto('/users')
    await this.waitForPageLoad()
  }

  async search(keyword: string) {
    await this.searchInput.fill(keyword)
    await this.page.keyboard.press('Enter')
    await this.waitForPageLoad()
  }

  async clickCreate() {
    await this.btnCreate.click()
  }

  async getRowCount() {
    return this.tableRows.count()
  }

  async clickEditButton(rowIndex = 0) {
    await this.tableRows.nth(rowIndex).getByRole('button', { name: '수정' }).click()
  }

  async clickDeleteButton(rowIndex = 0) {
    await this.tableRows.nth(rowIndex).getByRole('button', { name: '삭제' }).click()
  }

  async expectEmptyState() {
    await expect(this.page.getByText('조회된 데이터가 없습니다.')).toBeVisible()
  }
}
```

## Page Object 예시 (`tests/pages/UserCreatePage.ts`)

```typescript
import { type Page, type Locator } from '@playwright/test'
import { BasePage } from './BasePage'

export class UserCreatePage extends BasePage {
  readonly inputUserNm: Locator
  readonly inputEmail: Locator
  readonly btnSubmit: Locator
  readonly btnCancel: Locator

  constructor(page: Page) {
    super(page)
    this.inputUserNm = page.getByTestId('input-user-nm')
    this.inputEmail = page.getByTestId('input-email')
    this.btnSubmit = page.getByTestId('btn-submit')
    this.btnCancel = page.getByRole('button', { name: '취소' })
  }

  async goto() {
    await this.page.goto('/users/create')
    await this.waitForPageLoad()
  }

  async fillForm(data: { userNm: string; email: string }) {
    await this.inputUserNm.fill(data.userNm)
    await this.inputEmail.fill(data.email)
  }

  async submit() {
    await this.btnSubmit.click()
  }

  async cancel() {
    await this.btnCancel.click()
  }
}
```

## 테스트 파일 예시 (`tests/user-list.spec.ts`)

```typescript
import { test, expect } from '@playwright/test'
import { UserListPage } from './pages/UserListPage'
import { UserCreatePage } from './pages/UserCreatePage'

test.describe('UC-002 사용자 목록 조회', () => {

  test.beforeEach(async ({ page }) => {
    // 로그인 상태 설정 (storageState 또는 직접 로그인)
    await page.goto('/login')
    await page.getByTestId('input-login-id').fill('admin')
    await page.getByTestId('input-password').fill('password123!')
    await page.getByTestId('btn-login').click()
    await page.waitForURL('/dashboard')
  })

  test('목록 페이지가 정상적으로 표시된다', async ({ page }) => {
    const listPage = new UserListPage(page)
    await listPage.goto()

    await expect(page.getByRole('heading', { name: '사용자 목록' })).toBeVisible()
    await expect(listPage.btnCreate).toBeEnabled()
    const rowCount = await listPage.getRowCount()
    expect(rowCount).toBeGreaterThan(0)
  })

  test('사용자명으로 검색하면 결과가 필터링된다', async ({ page }) => {
    const listPage = new UserListPage(page)
    await listPage.goto()

    await listPage.search('홍길동')
    const rowCount = await listPage.getRowCount()
    expect(rowCount).toBeGreaterThanOrEqual(1)
  })

  test('존재하지 않는 검색어로 검색하면 빈 상태 메시지가 표시된다', async ({ page }) => {
    const listPage = new UserListPage(page)
    await listPage.goto()

    await listPage.search('존재하지않는사용자XYZ999')
    await listPage.expectEmptyState()
  })
})

test.describe('UC-003 사용자 등록', () => {

  test('필수 필드를 입력하면 사용자가 정상 등록된다', async ({ page }) => {
    const createPage = new UserCreatePage(page)
    await createPage.goto()

    await createPage.fillForm({
      userNm: '테스트사용자',
      email: 'test@example.com',
    })
    await createPage.submit()

    // 등록 후 목록 페이지로 이동 확인
    await expect(page).toHaveURL('/users')
  })

  test('이메일 형식이 잘못되면 유효성 오류가 표시된다', async ({ page }) => {
    const createPage = new UserCreatePage(page)
    await createPage.goto()

    await createPage.fillForm({ userNm: '테스트', email: '잘못된이메일' })
    await createPage.submit()

    await expect(page.getByText('이메일 형식이 올바르지 않습니다.')).toBeVisible()
  })

  test('필수 필드를 비우면 유효성 오류가 표시된다', async ({ page }) => {
    const createPage = new UserCreatePage(page)
    await createPage.goto()

    await createPage.submit()  // 빈 폼 제출

    await expect(page.getByText('사용자명을 입력하세요.')).toBeVisible()
  })

  test('취소 버튼을 누르면 이전 페이지로 이동한다', async ({ page }) => {
    await page.goto('/users')
    const createPage = new UserCreatePage(page)
    await createPage.goto()

    await createPage.cancel()
    await expect(page).toHaveURL('/users')
  })
})
```

## 공통 인증 fixture (`tests/fixtures/auth.fixture.ts`)

```typescript
import { test as base, type Page } from '@playwright/test'

interface AuthFixtures {
  authenticatedPage: Page
}

export const test = base.extend<AuthFixtures>({
  authenticatedPage: async ({ page }, use) => {
    await page.goto('/login')
    await page.getByTestId('input-login-id').fill(process.env.TEST_USER ?? 'admin')
    await page.getByTestId('input-password').fill(process.env.TEST_PASSWORD ?? 'password123!')
    await page.getByTestId('btn-login').click()
    await page.waitForURL('/dashboard')
    await use(page)
  },
})

export { expect } from '@playwright/test'
```

## .env.example (Playwright 전용)

```
BASE_URL=http://localhost:3000
TEST_USER=admin
TEST_PASSWORD=password123!
```

## package.json 스크립트 추가

```json
{
  "scripts": {
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:debug": "PWDEBUG=1 playwright test",
    "test:e2e:report": "playwright show-report"
  }
}
```
