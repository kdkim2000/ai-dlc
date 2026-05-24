# VO/DTO Java 클래스 템플릿

## 엔터티 VO — 테이블 전체 매핑

```java
package {{basePackage}}.vo;

import lombok.*;
import jakarta.validation.constraints.*;
import java.io.Serializable;
import java.time.LocalDateTime;

/**
 * {{테이블_한글명}} VO
 * 테이블: {{테이블명}}
 */
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class {{도메인명}}VO implements Serializable {

    private static final long serialVersionUID = 1L;

    /** {{pk설명}} */
    private Long {{pk필드명}};

    /** {{필드설명1}} */
    @NotBlank(message = "{{필드설명1}}은(는) 필수입니다.")
    @Size(max = 100)
    private String {{필드명1}};

    /** {{필드설명2}} */
    private String {{필드명2}};

    /** 생성일시 */
    private LocalDateTime createdAt;

    /** 수정일시 */
    private LocalDateTime updatedAt;

    /** 생성자ID */
    private String createdBy;
}
```

---

## 요청 VO (ReqVO) — API 요청 바디

```java
package {{basePackage}}.vo;

import lombok.*;
import jakarta.validation.constraints.*;

@Getter
@Setter
@NoArgsConstructor
public class {{도메인명}}ReqVO {

    @NotBlank(message = "{{필드설명}}은(는) 필수입니다.")
    @Size(max = 100, message = "{{필드설명}}은(는) 최대 100자입니다.")
    private String {{필드명1}};

    @NotNull(message = "{{필드설명2}}은(는) 필수입니다.")
    private Long {{필드명2}};

    @Email(message = "이메일 형식이 올바르지 않습니다.")
    private String email;
}
```

---

## 검색 조건 VO (SearchVO) — 목록 조회 페이징

```java
package {{basePackage}}.vo;

import lombok.*;

@Getter
@Setter
@NoArgsConstructor
public class {{도메인명}}SearchVO {

    /** 검색 키워드 */
    private String keyword;

    /** 사용여부 (Y/N) */
    private String useYn;

    /** 페이지 번호 (1부터 시작) */
    private int page = 1;

    /** 페이지 크기 */
    private int size = 10;

    /** 정렬 컬럼 */
    private String sortBy = "created_at";

    /** 정렬 방향 (ASC/DESC) */
    private String sortDir = "DESC";

    public int getOffset() {
        return (page - 1) * size;
    }
}
```

---

## 공통 BaseVO — 공통 컬럼 부모 클래스

```java
package {{basePackage}}.vo;

import lombok.*;
import java.io.Serializable;
import java.time.LocalDateTime;

@Getter
@Setter
@NoArgsConstructor
public abstract class BaseVO implements Serializable {

    private static final long serialVersionUID = 1L;

    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    private String createdBy;
}
```

---

## 타입 매핑 참조

| DB 타입       | Java 타입         | Lombok 어노테이션 예시 |
|:------------|:----------------|:----------------|
| BIGINT (PK) | `Long`          | `private Long id;` |
| INT         | `Integer`       | `private Integer sortOrd;` |
| VARCHAR     | `String`        | `@NotBlank private String name;` |
| CHAR(1)     | `String`        | `private String useYn;` |
| DATETIME    | `LocalDateTime` | `private LocalDateTime createdAt;` |
| DATE        | `LocalDate`     | `private LocalDate birthDate;` |
| DECIMAL     | `BigDecimal`    | `private BigDecimal amount;` |
| TINYINT(1)  | `Boolean`       | `private Boolean activeYn;` |
| TEXT        | `String`        | `private String content;` |
