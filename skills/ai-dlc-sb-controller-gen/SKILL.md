---
name: ai-dlc-sb-controller-gen
description: AI-DLC 개발단계(Spring Boot) 스킬. Controller 레이어 Java 코드를 생성한다. "Controller 코드 생성", "컨트롤러 만들어줘", "REST API 컨트롤러 생성", "API 엔드포인트 코드 생성", "RestController 만들어줘", "HTTP 핸들러 만들어줘", "컨트롤러 클래스 생성" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Controller 레이어 Java 코드 생성

API설계서(operationId, 경로, HTTP 메서드, 스키마)와 Service 인터페이스를 기반으로 **@RestController 클래스**를 실제 파일로 생성한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "Controller 코드 생성", "컨트롤러 만들어줘", "REST API 컨트롤러 생성"
- "API 엔드포인트 코드 생성", "RestController 만들어줘", "HTTP 핸들러 만들어줘"

## 입력

- **필수**: API설계서 (operationId, HTTP 메서드, 경로, 요청/응답 스키마)
- **선택**: Service 인터페이스 코드, 인증/인가 요구사항

## 분석 절차

1. **API 경로 그룹핑**: 동일 리소스 경로를 하나의 Controller 클래스로 묶음
2. **@RequestMapping 설정**: 공통 기본 경로 (`/api/v1/{리소스}`)
3. **메서드별 매핑**:
   - operationId → 메서드명 (camelCase)
   - GET(목록) → `@GetMapping` + `{도메인명}SearchVO` 파라미터
   - GET(단건) → `@GetMapping("/{id}")` + `@PathVariable Long id`
   - POST → `@PostMapping` + `@RequestBody @Valid {도메인명}ReqVO`
   - PUT → `@PutMapping("/{id}")` + `@RequestBody @Valid {도메인명}ReqVO`
   - DELETE → `@DeleteMapping("/{id}")` + `@PathVariable Long id`
4. **응답 포맷**: `ResponseEntity<ApiResponse<T>>` 래퍼 사용
5. **Swagger 어노테이션**: `@Operation(summary=)`, `@Tag(name=)`, `@ApiResponse`
6. **인증 처리**: `@PreAuthorize` 또는 `@Secured` (Spring Security 설정 연계)
7. **입력값 검증**: `@Valid` + `BindingResult` 처리
8. **파일 저장**: 실제 경로에 파일 생성

## 생성 원칙

- **Controller는 얇게**: 비즈니스 로직 금지, Service 호출만
- **@Valid 필수**: 요청 바디 + ReqVO Bean Validation 연계
- **ApiResponse 래퍼**: 모든 응답은 `{ code, message, data }` 형식
- **HTTP 상태코드**: 200(OK), 201(Created), 204(No Content), 400(Bad Request), 404(Not Found)
- **CORS 설정**: `@CrossOrigin` 대신 글로벌 `WebMvcConfigurer` 설정 권고

## 산출물

- `src/main/java/{basePackage}/controller/{도메인명}Controller.java`
