# 단위 테스트 코드 템플릿

## Service 단위 테스트 (Mockito)

```java
package {{basePackage}}.service;

import {{basePackage}}.mapper.{{도메인명}}Mapper;
import {{basePackage}}.service.impl.{{도메인명}}ServiceImpl;
import {{basePackage}}.vo.{{도메인명}}VO;
import {{basePackage}}.vo.{{도메인명}}SearchVO;
import org.junit.jupiter.api.*;
import org.junit.jupiter.api.extension.ExtendWith;
import org.mockito.*;
import org.mockito.junit.jupiter.MockitoExtension;
import java.util.*;
import static org.assertj.core.api.Assertions.*;
import static org.mockito.BDDMockito.*;

@ExtendWith(MockitoExtension.class)
@DisplayName("{{도메인명_한글}} 서비스 단위 테스트")
class {{도메인명}}ServiceTest {

    @Mock
    private {{도메인명}}Mapper {{도메인명소문자}}Mapper;

    @InjectMocks
    private {{도메인명}}ServiceImpl {{도메인명소문자}}Service;

    private {{도메인명}}VO sample;

    @BeforeEach
    void setUp() {
        sample = {{도메인명}}VO.builder()
            .{{pk필드명}}(1L)
            .{{필드명1}}("테스트값")
            .build();
    }

    @Nested
    @DisplayName("단건 조회")
    class GetOne {

        @Test
        @DisplayName("존재하는 ID 조회 시 VO 반환")
        void success() {
            given({{도메인명소문자}}Mapper.selectOne(1L)).willReturn(sample);

            {{도메인명}}VO result = {{도메인명소문자}}Service.get{{도메인명}}(1L);

            assertThat(result).isNotNull();
            assertThat(result.get{{pk필드명PascalCase}}()).isEqualTo(1L);
        }

        @Test
        @DisplayName("존재하지 않는 ID 조회 시 NoSuchElementException 발생")
        void notFound() {
            given({{도메인명소문자}}Mapper.selectOne(999L)).willReturn(null);

            assertThatThrownBy(() -> {{도메인명소문자}}Service.get{{도메인명}}(999L))
                .isInstanceOf(NoSuchElementException.class);
        }
    }

    @Nested
    @DisplayName("등록")
    class Create {

        @Test
        @DisplayName("유효한 VO 등록 시 Mapper insert 호출")
        void success() {
            given({{도메인명소문자}}Mapper.insert(any())).willReturn(1);

            {{도메인명소문자}}Service.create{{도메인명}}(sample);

            then({{도메인명소문자}}Mapper).should(times(1)).insert(any({{도메인명}}VO.class));
        }

        @Test
        @DisplayName("필수 값 NULL 입력 시 예외 발생")
        void nullInput() {
            {{도메인명}}VO invalid = new {{도메인명}}VO();
            // 비즈니스 검증 규칙에 따라 예외 타입 조정

            assertThatThrownBy(() -> {{도메인명소문자}}Service.create{{도메인명}}(invalid))
                .isInstanceOf(IllegalArgumentException.class);
        }
    }

    @Nested
    @DisplayName("삭제")
    class Delete {

        @Test
        @DisplayName("존재하는 ID 삭제 시 정상 처리")
        void success() {
            given({{도메인명소문자}}Mapper.selectOne(1L)).willReturn(sample);
            given({{도메인명소문자}}Mapper.delete(1L)).willReturn(1);

            {{도메인명소문자}}Service.delete{{도메인명}}(1L);

            then({{도메인명소문자}}Mapper).should(times(1)).delete(1L);
        }
    }
}
```

---

## Controller 단위 테스트 (MockMvc)

```java
package {{basePackage}}.controller;

import {{basePackage}}.service.{{도메인명}}Service;
import {{basePackage}}.vo.{{도메인명}}VO;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.junit.jupiter.api.*;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.boot.test.mock.mockito.MockBean;
import org.springframework.http.MediaType;
import org.springframework.test.web.servlet.MockMvc;
import java.util.*;
import static org.mockito.BDDMockito.*;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest({{도메인명}}Controller.class)
@DisplayName("{{도메인명_한글}} 컨트롤러 단위 테스트")
class {{도메인명}}ControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @MockBean
    private {{도메인명}}Service {{도메인명소문자}}Service;

    @Autowired
    private ObjectMapper objectMapper;

    @Test
    @DisplayName("GET /api/v1/{{리소스경로}} — 목록 조회 200 OK")
    void getList() throws Exception {
        given({{도메인명소문자}}Service.get{{도메인명}}List(any())).willReturn(List.of());

        mockMvc.perform(get("/api/v1/{{리소스경로}}")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.code").value("SUCCESS"));
    }

    @Test
    @DisplayName("GET /api/v1/{{리소스경로}}/{id} — 단건 조회 200 OK")
    void getOne() throws Exception {
        {{도메인명}}VO vo = {{도메인명}}VO.builder().{{pk필드명}}(1L).build();
        given({{도메인명소문자}}Service.get{{도메인명}}(1L)).willReturn(vo);

        mockMvc.perform(get("/api/v1/{{리소스경로}}/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.data.{{pk필드명}}").value(1));
    }

    @Test
    @DisplayName("POST /api/v1/{{리소스경로}} — 등록 201 Created")
    void create() throws Exception {
        String body = objectMapper.writeValueAsString(Map.of("{{필드명1}}", "테스트값"));

        mockMvc.perform(post("/api/v1/{{리소스경로}}")
                .contentType(MediaType.APPLICATION_JSON)
                .content(body))
            .andExpect(status().isCreated());
    }

    @Test
    @DisplayName("POST 시 필수값 누락 — 400 Bad Request")
    void createWithMissingField() throws Exception {
        String body = objectMapper.writeValueAsString(Map.of());

        mockMvc.perform(post("/api/v1/{{리소스경로}}")
                .contentType(MediaType.APPLICATION_JSON)
                .content(body))
            .andExpect(status().isBadRequest());
    }
}
```

---

## pom.xml — 테스트 의존성

```xml
<!-- Spring Boot Test (JUnit5 포함) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<!-- AssertJ (BDD 스타일 검증) — spring-boot-starter-test에 포함 -->
<!-- Mockito — spring-boot-starter-test에 포함 -->
```
