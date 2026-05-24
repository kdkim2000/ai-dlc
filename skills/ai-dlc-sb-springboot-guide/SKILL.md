---
name: ai-dlc-sb-springboot-guide
description: AI-DLC 개발단계(Spring Boot) 스킬. Spring Boot 개발 가이드라인을 제공한다. "Spring Boot 가이드", "스프링부트 개발 규칙", "Spring Boot 어떻게 설정해", "스프링부트 표준", "Spring Boot 컨벤션", "스프링부트 개발 방법", "Spring Boot 모범 사례" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Spring Boot 개발 가이드라인

Spring Boot 3.x 기반 프로젝트의 **구조·빈 생명주기·프로파일·예외 처리·응답 포맷** 가이드를 대화창에 직접 출력한다. 파일을 저장하지 않는다.

## 트리거

- "Spring Boot 가이드", "스프링부트 개발 규칙", "Spring Boot 어떻게 설정해"
- "스프링부트 표준", "Spring Boot 컨벤션", "스프링부트 개발 방법"

## 출력 형식

가이드라인을 대화창에 마크다운 형식으로 직접 출력한다. (파일 저장 없음)

---

## 가이드 내용 (인라인 내장)

### 1. 프로젝트 패키지 구조

```
{basePackage}/
├── controller/    # @RestController — HTTP 요청/응답 처리
├── service/       # @Service 인터페이스 + impl/ 구현체
├── mapper/        # @Mapper — DB 접근 (MyBatis)
├── vo/            # VO, ReqVO, SearchVO
├── config/        # @Configuration — 설정 클래스
├── exception/     # 커스텀 예외 + GlobalExceptionHandler
└── common/        # ApiResponse, 공통 유틸
```

**규칙**:
- 도메인 기준이 아닌 레이어 기준으로 패키지 구성 (소규모 프로젝트 기준)
- 대규모 프로젝트는 `{basePackage}/{도메인}/{레이어}` 구조 권장

### 2. 빈 등록과 의존성 주입

```java
// 올바름: 생성자 주입 (@RequiredArgsConstructor)
@Service
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {
    private final UserMapper userMapper;
}

// 금지: 필드 주입
@Autowired
private UserMapper userMapper; // 테스트 불가, 순환의존 감지 어려움
```

### 3. 프로파일(Profile) 설정

```yaml
# application.yml (공통)
spring:
  profiles:
    active: dev  # 기본값

# application-dev.yml
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/dev_db

# application-prod.yml
spring:
  datasource:
    url: jdbc:mysql://${DB_HOST}:3306/prod_db  # 환경변수 사용
```

**규칙**: 운영 환경 비밀번호·키는 환경변수(`${ENV_VAR}`)로 외부화, 소스코드에 하드코딩 금지

### 4. 공통 응답 포맷

```java
// 모든 API 응답은 ApiResponse<T> 래퍼 사용
public class ApiResponse<T> {
    private String code;    // "SUCCESS" | 오류코드
    private String message;
    private T data;
}

// 사용 예
return ResponseEntity.ok(ApiResponse.ok(userVO));
return ResponseEntity.badRequest().body(ApiResponse.error("INVALID_INPUT", "입력값 오류"));
```

### 5. 전역 예외 처리

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(NoSuchElementException.class)
    public ResponseEntity<ApiResponse<Void>> handleNotFound(NoSuchElementException e) {
        return ResponseEntity.status(404).body(ApiResponse.error("NOT_FOUND", e.getMessage()));
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiResponse<Void>> handleValidation(MethodArgumentNotValidException e) {
        String message = e.getBindingResult().getFieldErrors().stream()
            .map(f -> f.getField() + ": " + f.getDefaultMessage())
            .findFirst().orElse("입력값 오류");
        return ResponseEntity.badRequest().body(ApiResponse.error("INVALID_INPUT", message));
    }
}
```

### 6. 자동 설정 (Auto Configuration)

- Spring Boot는 `@SpringBootApplication`이 선언된 클래스 하위 패키지를 자동 스캔
- 커스텀 설정이 필요한 경우: `@Configuration` + `@Bean` 명시적 등록
- `@Conditional*` 어노테이션으로 프로파일별 조건부 빈 등록

### 7. Spring Boot 버전별 주요 변경사항

| 버전 | 주요 변경 |
|:---|:---|
| 3.x | Jakarta EE 전환 (`javax.*` → `jakarta.*`), Java 17 필수 |
| 3.x | Hibernate 6, Spring Security 6 |
| 2.x (구버전) | Java 8/11 지원, `javax.*` 패키지 |

> **이 프로젝트**: Spring Boot 3.x + Java 17 기준
