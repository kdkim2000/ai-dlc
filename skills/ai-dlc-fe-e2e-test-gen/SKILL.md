---
name: ai-dlc-fe-e2e-test-gen
description: AI-DLC 개발단계(프론트엔드-React) 스킬. Playwright 기반 e2e 테스트 코드를 생성한다. "e2e 테스트 만들어줘", "Playwright 테스트 생성", "통합 테스트 코드 생성", "브라우저 테스트 만들어줘", "E2E 시나리오 코드 만들어줘", "Playwright 스크립트 생성", "화면 자동화 테스트 만들어줘" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Playwright e2e 테스트 코드 생성

화면설계서(SCR-NNN)·유즈케이스(UC-NNN)·구현된 컴포넌트를 분석하여 Playwright 기반 e2e 테스트 코드를 생성한다. Page Object Model 패턴을 적용한다.
공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "e2e 테스트 만들어줘", "Playwright 테스트 생성", "통합 테스트 코드 생성"
- "브라우저 테스트 만들어줘", "E2E 시나리오 코드 만들어줘", "Playwright 스크립트 생성"
- "화면 자동화 테스트 만들어줘", "end-to-end 테스트 작성"

---

## 입력

### 필수
- 화면설계서(SCR-NNN) — 테스트 대상 화면 식별
- 유즈케이스(UC-NNN) — 주요 시나리오 도출

### 선택
- 구현된 컴포넌트 코드 (`src/pages/`) — data-testid 속성 확인
- 기존 Page Object 파일 — 재사용 여부 확인

---

## 분석 절차

### 1단계: 테스트 대상 화면 및 시나리오 도출
UC별 주요 시나리오를 3가지 유형으로 분류:
- **Happy Path**: 정상적인 사용자 흐름 (성공 케이스)
- **경계값**: 입력값 최소/최대, 필수 필드 빈값 등
- **에러 시나리오**: 서버 오류, 유효성 실패, 권한 없음

### 2단계: playwright.config.ts 생성/갱신
- `baseURL`: 개발 서버 URL (기본: `http://localhost:3000`)
- `testDir`: `tests/`
- `reporter`: html (기본), CI 환경 시 github 리포터 추가
- `use.screenshot: 'only-on-failure'`, `use.video: 'retain-on-failure'`

### 3단계: Page Object 클래스 설계 (`tests/pages/`)
각 화면마다 Page Object 클래스 생성:
```typescript
class XxxListPage {
  async goto() { ... }
  async search(keyword: string) { ... }
  async clickCreate() { ... }
  async getRowCount() { ... }
}
```
선택자 우선순위: `data-testid` > `role` > `text` > CSS

### 4단계: 테스트 파일 생성 (`tests/`)
각 UC 또는 화면별 `.spec.ts` 파일 생성:
```typescript
test.describe('UC-002 사용자 목록 조회', () => {
  test('목록이 정상적으로 표시된다', async ({ page }) => { ... })
  test('검색어로 필터링된다', async ({ page }) => { ... })
  test('빈 결과일 때 안내 메시지가 표시된다', async ({ page }) => { ... })
})
```

### 5단계: 공통 fixtures/helpers 생성
- `tests/fixtures/auth.fixture.ts`: 로그인 상태 공통 setup
- `tests/helpers/`: 공통 유틸리티 함수

---

## 코드 생성 원칙

- **Page Object Model**: 화면당 1개 Page Object 클래스 (`tests/pages/`)
- **data-testid 우선**: 선택자는 `data-testid` 속성 사용 (CSS 클래스 선택자 지양)
- **await 필수**: 모든 비동기 작업에 await 적용, 암묵적 대기 의존 금지
- **테스트 독립성**: 각 test()는 상태를 공유하지 않음 (`test.beforeEach`로 초기화)
- **시나리오 명칭**: 한국어 displayName 사용 (`test('사용자 목록이 표시된다', ...)`)
- **인증 처리**: `beforeEach`에서 로그인 상태 설정 (storageState 활용)

---

## 산출물

| 파일 경로 | 설명 |
|:---|:---|
| `playwright.config.ts` | Playwright 설정 (초기 생성 또는 갱신) |
| `tests/pages/{Xxx}Page.ts` | Page Object 클래스 |
| `tests/{화면명 또는 UC명}.spec.ts` | 테스트 시나리오 |
| `tests/fixtures/auth.fixture.ts` | 공통 인증 fixture |

template.md에서 각 파일의 기본 코드 골격을 참조한다.
