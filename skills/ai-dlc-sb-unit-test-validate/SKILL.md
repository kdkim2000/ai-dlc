---
name: ai-dlc-sb-unit-test-validate
description: AI-DLC 개발단계(Spring Boot) 스킬. 단위 테스트 코드의 품질을 검증한다. "테스트 코드 검토", "테스트 품질 확인", "테스트 커버리지 확인", "JUnit 테스트 리뷰", "단위 테스트 검증", "테스트 코드 리뷰해줘", "테스트 품질 점검" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC 단위 테스트 코드 품질 검증

생성된 단위 테스트 코드의 **커버리지 기준·테스트 독립성·Mock 적절성·경계값 누락·어노테이션 오용**을 정적 분석으로 검증하고, VI-NNN 이슈 목록과 종합 판정을 보고한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "테스트 코드 검토", "테스트 품질 확인", "테스트 커버리지 확인"
- "JUnit 테스트 리뷰", "단위 테스트 검증", "테스트 코드 리뷰해줘"

## 입력

- **필수**: 단위 테스트 코드 파일 (`src/test/java/...` 경로)
- **선택**: 대상 소스 코드 (Service/Controller), 비즈니스 규칙 문서

## 검증 항목

### 1. 구조·어노테이션 검증

| 코드 | 항목 | 기준 |
|:---|:---|:---|
| TA-001 | @ExtendWith 누락 | Service 테스트에 `@ExtendWith(MockitoExtension.class)` 필수 |
| TA-002 | @WebMvcTest 대상 오류 | Controller 테스트는 `@WebMvcTest`, 전체 컨텍스트 로드 금지 |
| TA-003 | @DisplayName 누락 | 모든 @Test 메서드에 한글 `@DisplayName` 필수 |
| TA-004 | @BeforeEach 초기화 누락 | 공통 픽스처는 `@BeforeEach`로 설정 |

### 2. 커버리지 기준 검증

| 코드 | 항목 | 기준 |
|:---|:---|:---|
| CO-001 | Service 커버리지 미달 | 공개 메서드 80% 이상 테스트 케이스 존재 여부 확인 |
| CO-002 | Controller 커버리지 미달 | HTTP 메서드별 정상·오류 케이스 존재 여부 |
| CO-003 | 경계값 테스트 누락 | null, 빈 값, 최대/최소값 케이스 누락 |
| CO-004 | 예외 케이스 누락 | `assertThrows` 또는 `andExpect(status().is4xxClientError())` 누락 |

### 3. Mock 적절성 검증

| 코드 | 항목 | 기준 |
|:---|:---|:---|
| MO-001 | Mock 과다 의존 | 테스트에서 실제 로직 없이 Mock 결과만 검증하는 경우 |
| MO-002 | verify() 남용 | 결과값 검증 없이 `verify(mock, times(N))` 만 존재 |
| MO-003 | @MockBean 오남용 | `@WebMvcTest`에서 불필요한 Bean 모킹 |

### 4. 독립성 검증

| 코드 | 항목 | 기준 |
|:---|:---|:---|
| IN-001 | 테스트 간 상태 공유 | static 필드에 공유 상태 존재 여부 |
| IN-002 | 실행 순서 의존 | `@TestMethodOrder` 없이 순서 의존적인 테스트 |

## 보고서 형식

```
## 단위 테스트 검증 보고서

| 항목 | 결과 |
|:---|:---|
| 검토 일시 | {{작성일시}} |
| 검토 파일 | {{파일목록}} |
| 총 이슈 수 | {{N}}건 |
| 종합 판정 | ✅ 적합 / ⚠️ 조건부 / ❌ 부적합 |

### 이슈 목록

| 이슈ID | 분류 | 파일명:라인 | 설명 | 수정 방안 |
|:---|:---|:---|:---|:---|
| VI-001 | CO-003 | UserServiceTest.java:45 | null 입력 케이스 누락 | assertThrows 추가 |
```

## 산출물

파일명: `테스트코드_검증_{YYYYMMDD}.md` (프로젝트 `docs/` 하위)
