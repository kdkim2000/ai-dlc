---
name: ai-dlc-sb-service-gen
description: AI-DLC 개발단계(Spring Boot) 스킬. Service 레이어 Java 코드를 생성한다. "Service 코드 생성", "서비스 레이어 만들어줘", "비즈니스 로직 생성", "Service 클래스 만들어줘", "서비스 구현체 생성", "ServiceImpl 만들어줘", "비즈니스 로직 코드 만들어줘" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Service 레이어 Java 코드 생성

클래스설계서(CLS-NNN Service)와 API설계서(operationId)를 기반으로 **Service 인터페이스 + ServiceImpl 클래스**를 실제 파일로 생성한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "Service 코드 생성", "서비스 레이어 만들어줘", "비즈니스 로직 생성"
- "Service 클래스 만들어줘", "ServiceImpl 만들어줘", "서비스 구현체 생성"

## 입력

- **필수**: 클래스설계서 (CLS-NNN Service 계층 정의) 또는 Mapper 인터페이스 코드
- **선택**: API설계서 (operationId → 메서드 시그니처 연계), 비즈니스 규칙 문서

## 분석 절차

1. **인터페이스 메서드 목록 결정**:
   - API설계서 operationId → 서비스 메서드명 매핑
   - CLS 클래스설계서 Service 메서드 목록 참조
   - CRUD 기본: `get{도메인명}List`, `get{도메인명}`, `create{도메인명}`, `update{도메인명}`, `delete{도메인명}`
2. **반환 타입 결정**:
   - 목록: `List<{도메인명}VO>` 또는 `Page<{도메인명}VO>`
   - 단건: `{도메인명}VO`
   - 등록/수정: `void` 또는 `{도메인명}VO`
3. **@Transactional 적용**:
   - 조회: `@Transactional(readOnly = true)`
   - 등록/수정/삭제: `@Transactional`
4. **예외 처리 설계**:
   - 존재하지 않는 리소스: `EntityNotFoundException`
   - 중복 데이터: `DuplicateKeyException`
   - 비즈니스 규칙 위반: 커스텀 예외 클래스
5. **비즈니스 검증 로직 삽입**: BR 문서 기반 유효성 검사 코드
6. **파일 저장**: 실제 경로에 파일 생성

## 생성 원칙

- **인터페이스/구현체 분리**: `{도메인명}Service` 인터페이스 + `{도메인명}ServiceImpl` 구현체
- **@RequiredArgsConstructor**: 생성자 주입 방식 (필드 주입 금지)
- **로깅**: 중요 비즈니스 이벤트에 `log.info()` 추가
- **검증 우선**: 비즈니스 검증을 DB 호출 전에 수행
- **SRP 준수**: 하나의 서비스 클래스에 메서드 10개 초과 시 분리 권고

## 산출물

- `src/main/java/{basePackage}/service/{도메인명}Service.java`
- `src/main/java/{basePackage}/service/impl/{도메인명}ServiceImpl.java`
