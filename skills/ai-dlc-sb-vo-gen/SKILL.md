---
name: ai-dlc-sb-vo-gen
description: AI-DLC 개발단계(Spring Boot) 스킬. VO/DTO 레이어 Java 코드를 생성한다. "VO 코드 생성", "DTO 만들어줘", "Value Object 생성", "VO 클래스 만들어줘", "데이터 객체 코드 생성", "도메인 객체 만들어줘", "엔터티 클래스 생성" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC VO/DTO 레이어 Java 코드 생성

데이터설계서(테이블 컬럼 정의)와 클래스설계서(CLS-NNN Domain/VO)를 기반으로 **Lombok 기반 VO/DTO Java 클래스**를 실제 파일로 생성한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "VO 코드 생성", "DTO 만들어줘", "Value Object 생성"
- "VO 클래스 만들어줘", "데이터 객체 코드 생성", "도메인 객체 만들어줘"

## 입력

- **필수**: 데이터설계서 (테이블명, 컬럼명, 타입, NOT NULL 여부)
- **선택**: 클래스설계서 (CLS-NNN Domain/VO 클래스 정의), 기존 VO 파일 경로

## 분석 절차

1. **테이블→클래스 매핑 결정**:
   - 테이블명 → 클래스명 (TB_ 접두사 제거, PascalCase 변환)
   - 컬럼명 → 필드명 (snake_case → camelCase)
2. **Java 타입 결정**:
   - VARCHAR/TEXT → `String`
   - INT/BIGINT(PK·FK) → `Long` / `Integer`
   - DATETIME → `LocalDateTime`
   - TINYINT(1) → `Boolean`
   - DECIMAL → `BigDecimal`
3. **Lombok 어노테이션 적용**:
   - `@Getter @Setter @NoArgsConstructor @AllArgsConstructor @Builder`
   - 검색 조건 VO → `@Getter @Setter @NoArgsConstructor`
4. **Bean Validation 어노테이션**:
   - NOT NULL 컬럼 → `@NotNull` / `@NotBlank`
   - 길이 제한 → `@Size(max=N)`
   - 이메일 형식 → `@Email`
5. **공통 컬럼 처리**: created_at/updated_at/created_by는 VO에 포함하되 @JsonIgnore 필요 여부 판단
6. **파일 저장**: 프로젝트 패키지 경로에 실제 파일 생성

## VO 클래스 분류

| 분류 | 용도 | 특징 |
|:---|:---|:---|
| 엔터티 VO | DB 테이블 1:1 매핑 | 모든 컬럼 포함 |
| 요청 VO (ReqVO) | API 요청 바디 | @NotNull/검증 어노테이션 |
| 응답 VO (ResVO) | API 응답 바디 | @JsonProperty 필요 시 |
| 검색 VO (SearchVO) | 목록 조회 조건 | 페이징 필드(page, size) 포함 |

## 생성 원칙

- **Lombok 필수**: getter/setter 직접 작성 금지
- **Builder 패턴**: 4개 이상 필드가 있는 VO에는 `@Builder` 적용
- **null 안전**: 컬럼 NOT NULL → `@NotNull` 또는 `@NotBlank` (String)
- **직렬화 고려**: `implements Serializable` 및 `serialVersionUID` 포함
- **공통 부모 클래스**: `BaseVO` 상속 여부는 프로젝트 관례에 따름

## 산출물

- `src/main/java/{basePackage}/vo/{도메인명}VO.java`
- `src/main/java/{basePackage}/vo/{도메인명}ReqVO.java` (요청용, 필요 시)
- `src/main/java/{basePackage}/vo/{도메인명}SearchVO.java` (검색 조건, 필요 시)
