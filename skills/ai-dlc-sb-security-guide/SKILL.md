---
name: ai-dlc-sb-security-guide
description: AI-DLC 개발단계(Spring Boot) 스킬. 보안 코딩 가이드라인을 제공한다. "보안 코딩 가이드", "시큐어 코딩", "SQL Injection 방지", "XSS 방지", "보안 규칙", "보안 코딩 방법", "취약점 방지", "OWASP 대응" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC 보안 코딩 가이드라인

Spring Boot 환경에서 OWASP Top 10 대응·SQL Injection 방지·XSS 방지·인증/인가·민감 데이터 처리·입력값 검증 방법을 대화창에 직접 출력한다. 파일을 저장하지 않는다.

## 트리거

- "보안 코딩 가이드", "시큐어 코딩", "SQL Injection 방지"
- "XSS 방지", "보안 규칙", "취약점 방지", "OWASP 대응"

---

## 가이드 내용 (인라인 내장)

### 1. SQL Injection 방지

```xml
<!-- 절대 금지: ${}는 String 치환 → SQL Injection 가능 -->
WHERE user_nm = '${userNm}'

<!-- 필수: #{}는 PreparedStatement → SQL Injection 방지 -->
WHERE user_nm = #{userNm}

<!-- 동적 ORDER BY: 화이트리스트 검증 후 ${}만 허용 -->
```

```java
// Service에서 정렬 컬럼 화이트리스트 검증
private static final Set<String> ALLOWED_SORT = Set.of("user_nm", "created_at", "dept_cd");

public List<UserVO> getUserList(UserSearchVO vo) {
    if (!ALLOWED_SORT.contains(vo.getSortBy())) {
        vo.setSortBy("created_at");
    }
    return userMapper.selectList(vo);
}
```

### 2. XSS 방지

```java
// 응답 헤더 설정 (Spring Security)
http.headers(h -> h
    .contentTypeOptions(Customizer.withDefaults())
    .xssProtection(Customizer.withDefaults())
    .frameOptions(f -> f.deny())
);

// 사용자 입력값 HTML 이스케이프 (Thymeleaf: 자동, JSON: 자동)
// 직접 HTML 렌더링 시: HtmlUtils.htmlEscape(input) 사용
```

### 3. 인증/인가

```java
// Controller 메서드 레벨 보안
@PreAuthorize("hasRole('ADMIN')")
@DeleteMapping("/{id}")
public ResponseEntity<?> delete(@PathVariable Long id) { ... }

// 권한 계층: ROLE_ADMIN > ROLE_MANAGER > ROLE_USER
// application.yml 또는 SecurityConfig에서 경로별 권한 설정
http.authorizeHttpRequests(a -> a
    .requestMatchers("/api/admin/**").hasRole("ADMIN")
    .requestMatchers("/api/**").authenticated()
    .anyRequest().permitAll()
);
```

### 4. 민감 데이터 처리

```java
// 비밀번호: BCrypt 해시 사용
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}

// 비밀번호 응답/로그 노출 금지
@JsonProperty(access = JsonProperty.Access.WRITE_ONLY)
private String password;

// 로그에 민감 정보 출력 금지
log.info("User login: {}", user.getUserNm());  // OK
log.info("Password: {}", user.getPassword());  // 절대 금지
```

### 5. 입력값 검증

```java
// Bean Validation (jakarta.validation)
@NotBlank(message = "사용자명은 필수입니다.")
@Size(max = 50, message = "사용자명은 50자 이하입니다.")
private String userNm;

@Email(message = "이메일 형식이 올바르지 않습니다.")
private String email;

@Pattern(regexp = "^[YN]$", message = "사용여부는 Y 또는 N만 허용됩니다.")
private String useYn;

// Controller에서 @Valid 필수
public ResponseEntity<?> create(@Valid @RequestBody UserReqVO req) { ... }
```

### 6. CSRF 설정

```java
// REST API(Stateless): CSRF 비활성화
http.csrf(csrf -> csrf.disable());

// 세션 기반 웹: CSRF 활성화 (기본값)
// Thymeleaf: <input type="hidden" th:name="${_csrf.parameterName}" th:value="${_csrf.token}"/>
```

### 7. 파일 업로드 보안

```java
// 허용 확장자 화이트리스트
private static final Set<String> ALLOWED_EXT = Set.of("jpg", "png", "pdf", "xlsx");

// 파일 크기 제한 (application.yml)
spring:
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 50MB
```

### 8. OWASP Top 10 대응 요약

| 순위 | 위협 | 핵심 대응 |
|:---|:---|:---|
| A01 | 접근 통제 실패 | `@PreAuthorize`, URL 레벨 권한 설정 |
| A02 | 암호화 실패 | BCrypt, HTTPS 강제, 민감 데이터 암호화 |
| A03 | Injection | `#{}` 사용, PreparedStatement, 입력 검증 |
| A05 | 보안 설정 오류 | Security Config 명시적 설정, 기본값 의존 금지 |
| A07 | 인증 실패 | 세션 고정 방지, 토큰 만료 설정 |
| A10 | SSRF | 외부 URL 화이트리스트 검증 |
