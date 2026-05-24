---
name: ai-dlc-sb-anyframe-guide
description: AI-DLC 개발단계(Spring Boot) 스킬. Anyframe 프레임워크 개발 가이드라인을 제공한다. "Anyframe 가이드", "애니프레임 개발 방법", "Anyframe 사용법", "Anyframe 컴포넌트 사용", "애니프레임 설정 방법", "Anyframe 어떻게 써", "애니프레임 규칙" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Anyframe 개발 가이드라인

Anyframe 기반 Spring Boot 프로젝트의 **공통 컴포넌트 활용·설정 방법·Spring Bean 통합**을 대화창에 직접 출력한다. 파일을 저장하지 않는다.

## 트리거

- "Anyframe 가이드", "애니프레임 개발 방법", "Anyframe 사용법"
- "Anyframe 컴포넌트 사용", "애니프레임 설정 방법", "Anyframe 어떻게 써"

---

## 가이드 내용 (인라인 내장)

### 1. Anyframe 개요

Anyframe은 전자정부 프레임워크 계열의 엔터프라이즈 Java 프레임워크로, Spring 위에서 동작하는 공통 컴포넌트(페이징·쿼리·공통 서비스)를 제공한다.

### 2. 의존성 설정

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.anyframe</groupId>
    <artifactId>anyframe-core</artifactId>
    <version>${anyframe.version}</version>
</dependency>
<dependency>
    <groupId>org.anyframe</groupId>
    <artifactId>anyframe-query-plugin</artifactId>
    <version>${anyframe.version}</version>
</dependency>
<dependency>
    <groupId>org.anyframe</groupId>
    <artifactId>anyframe-pagination-plugin</artifactId>
    <version>${anyframe.version}</version>
</dependency>
```

### 3. Anyframe 설정 클래스

```java
@Configuration
@ComponentScan(basePackages = "org.anyframe")
public class AnyframeConfig {

    @Bean
    public MessageSource messageSource() {
        ReloadableResourceBundleMessageSource ms = new ReloadableResourceBundleMessageSource();
        ms.setBasenames("classpath:message/message");
        ms.setDefaultEncoding("UTF-8");
        return ms;
    }
}
```

### 4. 페이징 컴포넌트 사용

```java
// Anyframe 페이징은 Page 객체를 통해 처리
import org.anyframe.pagination.Page;

// Service에서 페이징 처리
Page page = new Page(list, totalCount, currentPage, pageSize, pageUnit);
```

### 5. Anyframe QueryService 사용 (구형 DBIO 연동)

```java
@Autowired
private QueryService queryService;

// 목록 조회
List<Map<String, Object>> list = queryService.find("queryId.selectList", params);

// 단건 조회
Map<String, Object> result = queryService.findByPk("queryId.selectOne", pkValue);
```

### 6. 공통 서비스 컴포넌트

| 컴포넌트 | 역할 | 사용 위치 |
|:---|:---|:---|
| `MessageSource` | 다국어 메시지 | 예외 메시지, 검증 메시지 |
| `CryptoService` | 암호화/복호화 | 비밀번호, 민감 정보 |
| `IdGenService` | 채번(ID 생성) | PK, 문서번호 자동 생성 |
| `PaginationManager` | 페이징 UI | 목록 페이지 |

### 7. Spring Boot와 통합 시 주의사항

- Anyframe XML 설정과 Spring Boot 자동 설정이 충돌할 수 있음 → `@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class})` 필요 시 사용
- Anyframe QueryService와 MyBatis를 혼용하는 경우 DataSource 분리 또는 하나로 통일
- Spring Boot 3.x + Jakarta EE 환경에서는 Anyframe 구버전 호환성 검토 필요

### 8. 프로젝트 적용 체크리스트

- [ ] anyframe-* 의존성 pom.xml 추가
- [ ] `AnyframeConfig.java` 생성 및 `@Import`
- [ ] `message/` 폴더에 다국어 메시지 파일 준비
- [ ] QueryService 사용 시 query XML 파일 위치 설정
- [ ] 기동 시 Anyframe 컴포넌트 정상 등록 확인 (로그 확인)
