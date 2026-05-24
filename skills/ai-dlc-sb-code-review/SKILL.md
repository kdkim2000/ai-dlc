---
name: ai-dlc-sb-code-review
description: AI-DLC 개발단계(Spring Boot) 스킬. 생성된 코드의 품질을 검토한다. "코드 품질 검토", "생성된 코드 리뷰", "코드 검토해줘", "코딩 컨벤션 확인", "구현 코드 리뷰", "코드 품질 확인", "Spring Boot 코드 리뷰" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC 구현 코드 품질 검토

생성된 Spring Boot 소스코드를 대상으로 **가이드라인 준수·보안 취약점·코딩 컨벤션·레이어 의존 위반·성능 안티패턴**을 정적 분석하고, VI-NNN 이슈 목록과 종합 판정을 보고한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "코드 품질 검토", "생성된 코드 리뷰", "코드 검토해줘"
- "코딩 컨벤션 확인", "구현 코드 리뷰", "Spring Boot 코드 리뷰"

## 입력

- **필수**: 검토 대상 소스코드 파일 경로 (Controller/Service/Mapper/VO 중 하나 이상)
- **선택**: 클래스설계서, API설계서 (설계 대비 구현 일치 여부 검증용)

## 검증 항목

### 1. 레이어 컨벤션 (LC)

| 코드 | 항목 | 기준 |
|:---|:---|:---|
| LC-001 | Controller 비즈니스 로직 포함 | Controller는 Service 호출만, 비즈니스 로직 없어야 함 |
| LC-002 | Service에서 HttpServletRequest 참조 | Service는 웹 계층 의존 금지 |
| LC-003 | Mapper에서 Service 호출 | 역방향 의존 금지 |
| LC-004 | @Autowired 필드 주입 | `@RequiredArgsConstructor` 생성자 주입으로 변경 |
| LC-005 | 인터페이스 없는 ServiceImpl | Service 인터페이스 분리 필수 |

### 2. 보안 (SC)

| 코드 | 항목 | 기준 |
|:---|:---|:---|
| SC-001 | SQL Injection 위험 | MyBatis `${}` 사용 → `#{}` 로 변경 |
| SC-002 | XSS 취약 응답 | HTML 미이스케이프 문자열 직접 반환 |
| SC-003 | 인증 누락 | 보안 필요 엔드포인트에 `@PreAuthorize` 또는 Security Config 누락 |
| SC-004 | 민감 정보 노출 | 비밀번호·토큰을 로그·응답에 포함 |
| SC-005 | CSRF 미설정 | POST/PUT/DELETE에 CSRF 보호 미설정 |

### 3. 코딩 컨벤션 (CC)

| 코드 | 항목 | 기준 |
|:---|:---|:---|
| CC-001 | 명명 규칙 위반 | 클래스/메서드/변수 네이밍 컨벤션 미준수 |
| CC-002 | 로직 중복 | 동일 로직이 2개 이상 메서드에 중복 |
| CC-003 | 메서드 길이 초과 | 단일 메서드 50라인 초과 → 분리 권고 |
| CC-004 | 주석 부재 | 비즈니스 규칙·복잡 로직에 주석 없음 |

### 4. 성능 (PF)

| 코드 | 항목 | 기준 |
|:---|:---|:---|
| PF-001 | N+1 쿼리 패턴 | 반복문 내 개별 DB 조회 → JOIN/IN 쿼리 권고 |
| PF-002 | 대용량 결과 페이징 미적용 | 조건 없는 `selectAll()` 패턴 |
| PF-003 | @Transactional 범위 과대 | 불필요한 트랜잭션 범위 포함 |

## 보고서 형식

```markdown
## 코드 품질 검토 보고서

| 항목 | 결과 |
|:---|:---|
| 검토 일시 | {{작성일시}} |
| 검토 파일 수 | {{N}}개 |
| 총 이슈 수 | Critical {{N}} / Warning {{N}} / Info {{N}} |
| 종합 판정 | ✅ 적합 / ⚠️ 조건부 / ❌ 부적합 |

### 이슈 목록

| 이슈ID | 심각도 | 분류 | 파일명:라인 | 설명 | 수정 방안 |
|:---|:---|:---|:---|:---|:---|
| VI-001 | Critical | SC-001 | UserMapper.xml:23 | ${}로 SQL Injection 위험 | #{} 로 변경 |
| VI-002 | Warning | LC-004 | UserService.java:15 | @Autowired 필드 주입 | 생성자 주입으로 변경 |
```

## 산출물

파일명: `코드품질검토_{YYYYMMDD}.md` (프로젝트 `docs/` 하위)
