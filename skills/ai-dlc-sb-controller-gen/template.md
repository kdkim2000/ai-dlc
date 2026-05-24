# Controller 레이어 Java 코드 템플릿

## REST Controller 클래스

```java
package {{basePackage}}.controller;

import {{basePackage}}.service.{{도메인명}}Service;
import {{basePackage}}.vo.{{도메인명}}VO;
import {{basePackage}}.vo.{{도메인명}}ReqVO;
import {{basePackage}}.vo.{{도메인명}}SearchVO;
import {{basePackage}}.common.ApiResponse;
import io.swagger.v3.oas.annotations.Operation;
import io.swagger.v3.oas.annotations.Parameter;
import io.swagger.v3.oas.annotations.tags.Tag;
import jakarta.validation.Valid;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@Tag(name = "{{도메인명_한글}}", description = "{{도메인명_한글}} 관리 API")
@RestController
@RequestMapping("/api/v1/{{리소스경로}}")
@RequiredArgsConstructor
public class {{도메인명}}Controller {

    private final {{도메인명}}Service {{도메인명소문자}}Service;

    /**
     * 목록 조회 — {{operationId_list}}
     */
    @Operation(summary = "{{도메인명_한글}} 목록 조회")
    @GetMapping
    public ResponseEntity<ApiResponse<List<{{도메인명}}VO>>> getList(
            {{도메인명}}SearchVO searchVO) {
        List<{{도메인명}}VO> list = {{도메인명소문자}}Service.get{{도메인명}}List(searchVO);
        int total = {{도메인명소문자}}Service.get{{도메인명}}Count(searchVO);
        return ResponseEntity.ok(ApiResponse.ok(list));
    }

    /**
     * 단건 조회 — {{operationId_get}}
     */
    @Operation(summary = "{{도메인명_한글}} 단건 조회")
    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<{{도메인명}}VO>> getOne(
            @Parameter(description = "{{pk설명}}") @PathVariable Long id) {
        {{도메인명}}VO vo = {{도메인명소문자}}Service.get{{도메인명}}(id);
        return ResponseEntity.ok(ApiResponse.ok(vo));
    }

    /**
     * 등록 — {{operationId_create}}
     */
    @Operation(summary = "{{도메인명_한글}} 등록")
    @PostMapping
    public ResponseEntity<ApiResponse<Void>> create(
            @Valid @RequestBody {{도메인명}}ReqVO reqVO) {
        {{도메인명}}VO vo = new {{도메인명}}VO();
        // ReqVO → VO 매핑 (ModelMapper 또는 직접 매핑)
        vo.set{{필드명1PascalCase}}(reqVO.get{{필드명1PascalCase}}());
        {{도메인명소문자}}Service.create{{도메인명}}(vo);
        return ResponseEntity.status(HttpStatus.CREATED).body(ApiResponse.ok(null));
    }

    /**
     * 수정 — {{operationId_update}}
     */
    @Operation(summary = "{{도메인명_한글}} 수정")
    @PutMapping("/{id}")
    public ResponseEntity<ApiResponse<Void>> update(
            @PathVariable Long id,
            @Valid @RequestBody {{도메인명}}ReqVO reqVO) {
        {{도메인명}}VO vo = new {{도메인명}}VO();
        vo.set{{pk필드명PascalCase}}(id);
        vo.set{{필드명1PascalCase}}(reqVO.get{{필드명1PascalCase}}());
        {{도메인명소문자}}Service.update{{도메인명}}(vo);
        return ResponseEntity.ok(ApiResponse.ok(null));
    }

    /**
     * 삭제 — {{operationId_delete}}
     */
    @Operation(summary = "{{도메인명_한글}} 삭제")
    @DeleteMapping("/{id}")
    public ResponseEntity<ApiResponse<Void>> delete(
            @PathVariable Long id) {
        {{도메인명소문자}}Service.delete{{도메인명}}(id);
        return ResponseEntity.status(HttpStatus.NO_CONTENT).body(ApiResponse.ok(null));
    }
}
```

---

## ApiResponse 공통 응답 래퍼

```java
package {{basePackage}}.common;

import lombok.*;

@Getter
@NoArgsConstructor
@AllArgsConstructor
public class ApiResponse<T> {

    private String code;
    private String message;
    private T data;

    public static <T> ApiResponse<T> ok(T data) {
        return new ApiResponse<>("SUCCESS", "정상 처리되었습니다.", data);
    }

    public static <T> ApiResponse<T> error(String code, String message) {
        return new ApiResponse<>(code, message, null);
    }
}
```

---

## Swagger/OpenAPI 설정 (SpringDoc)

```java
package {{basePackage}}.config;

import io.swagger.v3.oas.models.OpenAPI;
import io.swagger.v3.oas.models.info.Info;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
public class SwaggerConfig {

    @Bean
    public OpenAPI openAPI() {
        return new OpenAPI()
            .info(new Info()
                .title("{{사업명}} API")
                .description("{{사업명}} REST API 명세")
                .version("v1.0.0"));
    }
}
```

---

## pom.xml — SpringDoc 의존성

```xml
<!-- Swagger UI (SpringDoc OpenAPI) -->
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```
