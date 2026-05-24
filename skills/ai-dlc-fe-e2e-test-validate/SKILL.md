---
name: ai-dlc-fe-e2e-test-validate
description: AI-DLC 개발단계(프론트엔드-React) 스킬. Playwright e2e 테스트 코드의 품질과 UC 커버리지를 검증한다. "e2e 테스트 검토해줘", "Playwright 테스트 리뷰", "통합 테스트 검증", "E2E 테스트 품질 확인", "e2e 테스트 코드 검토", "Playwright 테스트 점검", "브라우저 테스트 검토" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Playwright e2e 테스트 코드 검증

생성된 Playwright 테스트 코드의 UC 커버리지·Page Object 패턴·선택자 품질·비동기 처리·시나리오 완전성을 검증하고 보고서를 생성한다. 파일을 수정하지 않는다.

## 트리거

- "e2e 테스트 검토해줘", "Playwright 테스트 리뷰", "통합 테스트 검증"
- "E2E 테스트 품질 확인", "e2e 테스트 코드 검토", "Playwright 테스트 점검"
- "브라우저 테스트 검토", "E2E 테스트 코드 리뷰"

---

## 입력

### 필수
- e2e 테스트 디렉터리 (`tests/`)
- 화면설계서 또는 유즈케이스 목록 (UC 커버리지 확인용)

---

## 검증 항목

### EV-001: UC 커버리지 (핵심 시나리오 누락)
- **검사**: 유즈케이스(UC-NNN) 별로 Happy Path 테스트가 존재하는지 확인
- **기준**: 우선순위 High UC의 Happy Path ≥ 100%, Medium UC ≥ 70%
- **위반 예**: UC-003 사용자 등록에 대한 정상 등록 테스트 없음

### EV-002: Page Object 패턴 미준수
- **검사**: `tests/pages/` 디렉터리 존재 여부, 각 화면별 Page Object 클래스 존재 여부
- **위반 예**: 테스트 파일에서 직접 `page.locator('.user-name')` 반복 사용

### EV-003: 하드코딩 선택자 사용
```typescript
// 위반 — CSS 클래스 기반 선택자 (변경에 취약)
page.locator('.btn-primary')
page.locator('#user-form > div > input')

// 권장 — data-testid, role, text 기반
page.getByTestId('btn-submit')
page.getByRole('button', { name: '등록' })
page.getByLabel('사용자명')
```

### EV-004: 비동기 처리 문제
```typescript
// 위반 — await 누락
page.click(page.getByTestId('btn-submit'))  // await 없음

// 위반 — 명시적 sleep 사용 (권장: waitFor 사용)
await page.waitForTimeout(3000)  // 안티패턴

// 권장
await page.getByTestId('btn-submit').click()
await page.waitForURL('/users')
await expect(page.getByText('등록 완료')).toBeVisible()
```

### EV-005: 테스트 독립성 미확보
- **검사**: `test.beforeEach` 없이 이전 test의 상태 의존 여부
- **위반 예**: 등록 테스트 이후 상태를 목록 테스트에서 기대

### EV-006: 에러 시나리오 누락
- **검사**: FORM 화면에 유효성 실패·서버 오류 시나리오가 있는지 확인
- **기준**: FORM 화면 테스트마다 최소 1개 에러 시나리오

### EV-007: playwright.config.ts 설정 누락
- `baseURL` 설정 여부
- `screenshot: 'only-on-failure'` 설정 여부
- CI 환경 대응 (`retries`, `reporter`) 설정 여부

---

## 검증 방법

1. `Glob`으로 `tests/**/*.spec.ts` 파일 목록 수집
2. `Glob`으로 `tests/pages/*.ts` Page Object 클래스 목록 수집
3. `Read`로 `playwright.config.ts` 읽어 설정 확인
4. `Grep`으로 패턴 검색:
   - `waitForTimeout` — EV-004 sleep 안티패턴
   - `locator\('\.` — EV-003 CSS 클래스 선택자
   - `await` 누락 패턴 (`.click()`, `.fill()` 앞 await 여부)
5. UC 목록 vs spec.ts의 `test.describe` 매핑 확인

---

## 보고서 형식

```markdown
# e2e 테스트 코드 검증 보고서

| 항목 | 내용 |
|:---|:---|
| 검증일 | YYYY-MM-DD |
| 검증 범위 | tests/ |

## UC 커버리지

| UC-ID | UC명 | 우선순위 | Happy Path | 에러 시나리오 | 판정 |
|:---|:---|:---:|:---:|:---:|:---:|
| UC-002 | 사용자 목록 조회 | High | ✅ | ✅ | 통과 |
| UC-003 | 사용자 등록 | High | ✅ | ❌ | 미흡 |
| UC-004 | 사용자 수정 | Medium | ❌ | ❌ | 누락 |

## 이슈 목록

| VI-ID | 코드 | 파일 | 라인 | 설명 | 심각도 |
|:---|:---|:---|:---:|:---|:---:|
| VI-001 | EV-003 | tests/user-list.spec.ts | 22 | CSS 클래스 선택자 사용 — data-testid로 변경 필요 | 중간 |
| VI-002 | EV-006 | tests/user-create.spec.ts | — | 등록 에러 시나리오(유효성 실패) 누락 | 중간 |

**종합 판정**: 통과 / 조건부통과 / 재검토필요
```

---

## 산출물

- `e2e테스트_검증_{YYYYMMDD}.md`
