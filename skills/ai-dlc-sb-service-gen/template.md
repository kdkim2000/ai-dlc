# Service 레이어 Java 코드 템플릿

## Service 인터페이스

```java
package {{basePackage}}.service;

import {{basePackage}}.vo.{{도메인명}}VO;
import {{basePackage}}.vo.{{도메인명}}SearchVO;
import java.util.List;

public interface {{도메인명}}Service {

    /** 목록 조회 */
    List<{{도메인명}}VO> get{{도메인명}}List({{도메인명}}SearchVO searchVO);

    /** 전체 건수 */
    int get{{도메인명}}Count({{도메인명}}SearchVO searchVO);

    /** 단건 조회 */
    {{도메인명}}VO get{{도메인명}}(Long {{pk필드명}});

    /** 등록 */
    void create{{도메인명}}({{도메인명}}VO vo);

    /** 수정 */
    void update{{도메인명}}({{도메인명}}VO vo);

    /** 삭제 */
    void delete{{도메인명}}(Long {{pk필드명}});
}
```

---

## ServiceImpl 구현체

```java
package {{basePackage}}.service.impl;

import {{basePackage}}.mapper.{{도메인명}}Mapper;
import {{basePackage}}.service.{{도메인명}}Service;
import {{basePackage}}.vo.{{도메인명}}VO;
import {{basePackage}}.vo.{{도메인명}}SearchVO;
import lombok.RequiredArgsConstructor;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;
import java.util.List;
import java.util.NoSuchElementException;

@Slf4j
@Service
@RequiredArgsConstructor
public class {{도메인명}}ServiceImpl implements {{도메인명}}Service {

    private final {{도메인명}}Mapper {{도메인명소문자}}Mapper;

    @Override
    @Transactional(readOnly = true)
    public List<{{도메인명}}VO> get{{도메인명}}List({{도메인명}}SearchVO searchVO) {
        return {{도메인명소문자}}Mapper.selectList(searchVO);
    }

    @Override
    @Transactional(readOnly = true)
    public int get{{도메인명}}Count({{도메인명}}SearchVO searchVO) {
        return {{도메인명소문자}}Mapper.count(searchVO);
    }

    @Override
    @Transactional(readOnly = true)
    public {{도메인명}}VO get{{도메인명}}(Long {{pk필드명}}) {
        {{도메인명}}VO vo = {{도메인명소문자}}Mapper.selectOne({{pk필드명}});
        if (vo == null) {
            throw new NoSuchElementException("{{도메인명_한글}} 정보를 찾을 수 없습니다. id=" + {{pk필드명}});
        }
        return vo;
    }

    @Override
    @Transactional
    public void create{{도메인명}}({{도메인명}}VO vo) {
        // 비즈니스 검증
        validate{{도메인명}}(vo);
        int result = {{도메인명소문자}}Mapper.insert(vo);
        log.info("{{도메인명_한글}} 등록 완료: id={}", vo.get{{pk필드명PascalCase}}());
    }

    @Override
    @Transactional
    public void update{{도메인명}}({{도메인명}}VO vo) {
        // 존재 여부 확인
        get{{도메인명}}(vo.get{{pk필드명PascalCase}}());
        int result = {{도메인명소문자}}Mapper.update(vo);
        log.info("{{도메인명_한글}} 수정 완료: id={}", vo.get{{pk필드명PascalCase}}());
    }

    @Override
    @Transactional
    public void delete{{도메인명}}(Long {{pk필드명}}) {
        // 존재 여부 확인
        get{{도메인명}}({{pk필드명}});
        {{도메인명소문자}}Mapper.delete({{pk필드명}});
        log.info("{{도메인명_한글}} 삭제 완료: id={}", {{pk필드명}});
    }

    /** 비즈니스 검증 */
    private void validate{{도메인명}}({{도메인명}}VO vo) {
        // BR-NNN: {{비즈니스규칙_설명}}
        // 예: if (vo.getXxx() == null) throw new IllegalArgumentException("...");
    }
}
```

---

## 커스텀 예외 클래스

```java
package {{basePackage}}.exception;

public class BusinessException extends RuntimeException {
    private final String errorCode;

    public BusinessException(String errorCode, String message) {
        super(message);
        this.errorCode = errorCode;
    }

    public String getErrorCode() {
        return errorCode;
    }
}
```

---

## GlobalExceptionHandler (Service 예외 처리)

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(NoSuchElementException.class)
    public ResponseEntity<ApiResponse<Void>> handleNotFound(NoSuchElementException e) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(ApiResponse.error("NOT_FOUND", e.getMessage()));
    }

    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ApiResponse<Void>> handleBusiness(BusinessException e) {
        return ResponseEntity.badRequest()
            .body(ApiResponse.error(e.getErrorCode(), e.getMessage()));
    }
}
```
