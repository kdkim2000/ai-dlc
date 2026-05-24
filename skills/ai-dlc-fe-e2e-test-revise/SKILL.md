---
name: ai-dlc-fe-e2e-test-revise
description: AI-DLC 개발단계(프론트엔드-React) 스킬. e2e 테스트 검증 결과를 반영하여 테스트 코드를 수정한다. "e2e 테스트 수정해줘", "Playwright 테스트 개선", "통합 테스트 보완", "E2E 테스트 커버리지 보완", "누락된 시나리오 추가", "테스트 선택자 수정", "e2e 테스트 고쳐줘" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC e2e 테스트 코드 수정

e2e 테스트 검증 보고서(`e2e테스트_검증_{YYYYMMDD}.md`)의 이슈 목록을 읽어 테스트 코드를 수정한다. 누락된 시나리오 추가, 하드코딩 선택자 교체, 비동기 처리 보완, Page Object 패턴 적용을 수행한다.

## 트리거

- "e2e 테스트 수정해줘", "Playwright 테스트 개선", "통합 테스트 보완"
- "E2E 테스트 커버리지 보완", "누락된 시나리오 추가"
- "테스트 선택자 수정", "e2e 테스트 고쳐줘", "Playwright 이슈 반영"

---

## 입력

### 필수
- e2e 테스트 검증 보고서 (`e2e테스트_검증_*.md`)

### 선택
- 특정 VI-ID 또는 EV 코드만 적용 (미지정 시 전체 적용)

---

## 수정 절차

1. `Read`로 검증 보고서 읽어 이슈 목록(VI-NNN) 파악
2. 수정 우선순위:
   - **1순위 EV-001** — UC 커버리지 누락 (Happy Path 미존재)
   - **2순위 EV-004** — 비동기 처리 오류 (await 누락, sleep 안티패턴)
   - **3순위 EV-006** — 에러 시나리오 누락
   - **4순위 EV-003** — 하드코딩 선택자 교체
   - **5순위 EV-002** — Page Object 패턴 미준수
   - **6순위 EV-005** — 테스트 독립성 미확보
   - **7순위 EV-007** — playwright.config.ts 설정 보완
3. 각 이슈별로 `Read`로 대상 파일 확인
4. `Edit` 또는 `Write`로 수정 적용 (신규 시나리오 추가 시 기존 `test.describe` 블록 내 삽입)
5. 수정 결과 표 출력

---

## 주요 수정 패턴

### EV-001: 누락된 Happy Path 추가
```typescript
// tests/user-edit.spec.ts 에 Happy Path 추가
test.describe('UC-004 사용자 수정', () => {
  test('필수 필드를 수정하면 변경사항이 저장된다', async ({ page }) => {
    const editPage = new UserEditPage(page)
    await editPage.goto(1)
    await editPage.fillForm({ userNm: '수정된이름' })
    await editPage.submit()
    await expect(page).toHaveURL('/users')
    await expect(page.getByText('수정 완료')).toBeVisible()
  })
})
```

### EV-003: 하드코딩 선택자 교체
```typescript
// 위반
page.locator('.btn-primary')
page.locator('#user-form > div > input')

// 수정 — data-testid 기반
page.getByTestId('btn-submit')
page.getByTestId('input-user-nm')
```

### EV-004: await 누락 수정
```typescript
// 위반
page.click(page.getByTestId('btn-submit'))

// 수정
await page.getByTestId('btn-submit').click()
```

### EV-004: sleep 안티패턴 제거
```typescript
// 위반
await page.waitForTimeout(3000)

// 수정 — 명시적 대기로 교체
await page.waitForURL('/users')
// 또는
await expect(page.getByText('등록 완료')).toBeVisible()
```

### EV-005: beforeEach로 독립성 확보
```typescript
// 수정 — 각 테스트 전 초기 상태 설정
test.beforeEach(async ({ page }) => {
  await page.goto('/login')
  await page.getByTestId('input-login-id').fill('admin')
  await page.getByTestId('input-password').fill('password123!')
  await page.getByTestId('btn-login').click()
  await page.waitForURL('/dashboard')
})
```

### EV-006: 에러 시나리오 추가
```typescript
// FORM 화면에 에러 시나리오 추가
test('필수 필드를 비우면 유효성 오류가 표시된다', async ({ page }) => {
  const createPage = new UserCreatePage(page)
  await createPage.goto()
  await createPage.submit()  // 빈 폼 제출
  await expect(page.getByText('사용자명을 입력하세요.')).toBeVisible()
})

test('중복 이메일 등록 시 서버 오류 메시지가 표시된다', async ({ page }) => {
  const createPage = new UserCreatePage(page)
  await createPage.goto()
  await createPage.fillForm({ userNm: '테스트', email: 'existing@example.com' })
  await createPage.submit()
  await expect(page.getByText('이미 사용 중인 이메일입니다.')).toBeVisible()
})
```

### EV-007: playwright.config.ts 설정 보완
```typescript
// 누락 설정 추가
export default defineConfig({
  use: {
    baseURL: process.env.BASE_URL ?? 'http://localhost:3000',
    screenshot: 'only-on-failure',   // 누락 시 추가
    video: 'retain-on-failure',
  },
  retries: process.env.CI ? 2 : 0,  // CI 재시도
  reporter: process.env.CI
    ? [['github'], ['html', { open: 'never' }]]
    : [['html', { open: 'on-failure' }]],
})
```

---

## 수정 결과 표

```markdown
## e2e 테스트 수정 결과

| VI-ID | EV코드 | 파일 | 수정 내용 | 상태 |
|:---|:---|:---|:---|:---:|
| VI-001 | EV-001 | tests/user-edit.spec.ts | UC-004 Happy Path 테스트 추가 | ✅ 완료 |
| VI-002 | EV-003 | tests/user-list.spec.ts | CSS 선택자 → data-testid 교체 | ✅ 완료 |
| VI-003 | EV-006 | tests/user-create.spec.ts | 에러 시나리오 2건 추가 | ✅ 완료 |

**적용 완료**: N건 / **건너뜀**: N건
```

---

## 산출물

- 수정된 테스트 파일들 (`tests/**/*.spec.ts`, `tests/pages/*.ts`)
- 수정 결과 표 (대화창 출력)
