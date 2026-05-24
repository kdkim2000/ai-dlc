---
name: ai-dlc-sb-unit-test-gen
description: AI-DLC 개발단계(Spring Boot) 스킬. JUnit5/Mockito 기반 단위 테스트 코드를 생성한다. "단위 테스트 생성", "JUnit 테스트 만들어줘", "테스트 코드 생성", "unit test 작성", "모의 객체 테스트 만들어줘", "테스트 케이스 생성", "MockMvc 테스트 만들어줘" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC 단위 테스트 코드 생성 (JUnit5/Mockito)

생성된 Service/Controller/Mapper 코드를 기반으로 **JUnit5 + Mockito 기반 단위 테스트 클래스**를 실제 파일로 생성한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "단위 테스트 생성", "JUnit 테스트 만들어줘", "테스트 코드 생성"
- "unit test 작성", "모의 객체 테스트 만들어줘", "MockMvc 테스트 만들어줘"

## 입력

- **필수**: 대상 클래스 코드 (Service/Controller/Mapper 중 하나 이상)
- **선택**: 비즈니스 규칙 문서 (경계값·예외 케이스 도출)

## 분석 절차

1. **레이어별 테스트 전략 결정**:
   - ServiceImpl: `@ExtendWith(MockitoExtension.class)` + `@Mock Mapper`
   - Controller: `@WebMvcTest` + MockMvc + `@MockBean Service`
   - Mapper: `@MybatisTest` + 인메모리 DB (선택)
2. **테스트 케이스 목록 도출**:
   - 정상 케이스: 유효한 입력 → 기대 결과 검증
   - 경계값: null 입력, 빈 문자열, 최대/최소 값
   - 예외 케이스: 존재하지 않는 ID, 중복 데이터, 권한 없음
3. **Mock 설정**: `@Mock`, `@InjectMocks`, `when().thenReturn()` 설정
4. **검증 메서드**: `assertEquals`, `assertThrows`, `verify(mock, times(N))`
5. **테스트 명명**: `@DisplayName` 한글 설명 필수
6. **파일 저장**: `src/test/java/...` 경로에 실제 파일 생성

## 테스트 커버리지 기준

| 레이어 | 최소 커버리지 | 핵심 검증 항목 |
|:------|:----------:|:-----------|
| Service | ≥ 80% | 비즈니스 로직, 예외 처리, 트랜잭션 |
| Controller | ≥ 70% | HTTP 상태코드, 응답 포맷, 입력 검증 |
| Mapper | 선택 (통합 테스트 권장) | SQL 실행 결과 |

## 생성 원칙

- **독립성**: 각 테스트는 다른 테스트에 의존하지 않음 (`@BeforeEach` 초기화)
- **Mock 최소화**: 실제 로직이 있는 곳은 Mock 대신 실제 객체 사용
- **명확한 검증**: `verify()` 남용 금지, 결과값 직접 검증 우선
- **한글 @DisplayName**: `@DisplayName("등록 시 NULL 입력이면 예외 발생")` 형식

## 산출물

- `src/test/java/{basePackage}/service/{도메인명}ServiceTest.java`
- `src/test/java/{basePackage}/controller/{도메인명}ControllerTest.java`
