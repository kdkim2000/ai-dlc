# Anyframe 설정 구조

## pom.xml 추가 의존성

```xml
<!-- Anyframe Core -->
<dependency>
    <groupId>org.anyframe</groupId>
    <artifactId>anyframe-core</artifactId>
    <version>{{anyframe_version}}</version>  <!-- TODO: 버전 확인 필요 -->
</dependency>
<!-- Anyframe Query Plugin (MyBatis 연동) -->
<dependency>
    <groupId>org.anyframe</groupId>
    <artifactId>anyframe-query-plugin</artifactId>
    <version>{{anyframe_version}}</version>
</dependency>
<!-- Anyframe Pagination Plugin -->
<dependency>
    <groupId>org.anyframe</groupId>
    <artifactId>anyframe-pagination-plugin</artifactId>
    <version>{{anyframe_version}}</version>
</dependency>
```

---

## 공통 응답 포맷 (ApiResponse.java)

```java
package {{base_package}}.vo;

import lombok.Builder;
import lombok.Getter;

@Getter
@Builder
public class ApiResponse<T> {
    private boolean success;
    private String message;
    private T data;

    public static <T> ApiResponse<T> ok(T data) {
        return ApiResponse.<T>builder()
                .success(true)
                .message("OK")
                .data(data)
                .build();
    }

    public static <T> ApiResponse<T> error(String message) {
        return ApiResponse.<T>builder()
                .success(false)
                .message(message)
                .build();
    }
}
```

---

## 공통 예외 처리 (GlobalExceptionHandler.java)

```java
package {{base_package}}.exception;

import {{base_package}}.vo.ApiResponse;
import lombok.extern.slf4j.Slf4j;
import org.springframework.http.HttpStatus;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseStatus;
import org.springframework.web.bind.annotation.RestControllerAdvice;

@Slf4j
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiResponse<Void> handleValidationException(MethodArgumentNotValidException e) {
        log.warn("Validation error: {}", e.getMessage());
        return ApiResponse.error(e.getBindingResult().getFieldError().getDefaultMessage());
    }

    @ExceptionHandler(Exception.class)
    @ResponseStatus(HttpStatus.INTERNAL_SERVER_ERROR)
    public ApiResponse<Void> handleException(Exception e) {
        log.error("Unexpected error", e);
        return ApiResponse.error("서버 오류가 발생했습니다.");
    }
}
```

---

## Anyframe 설정 검증 체크리스트

| 항목 | 확인 여부 | 비고 |
|:---|:---:|:---|
| Anyframe 의존성 추가 | ☐ | pom.xml/build.gradle |
| 공통 응답 포맷 클래스 생성 | ☐ | ApiResponse.java |
| 공통 예외 처리 클래스 생성 | ☐ | GlobalExceptionHandler.java |
| 트랜잭션 매니저 설정 | ☐ | application.yml 또는 Config 클래스 |
| 페이지네이션 설정 | ☐ | 필요 시 |
| 애플리케이션 기동 테스트 | ☐ | 에러 없이 실행 확인 |
