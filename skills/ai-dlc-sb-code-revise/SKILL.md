---
name: ai-dlc-sb-code-revise
description: AI-DLC 개발단계(Spring Boot) 스킬. 코드 품질 검토 결과를 반영하여 소스코드를 수정한다. "코드 수정해줘", "리뷰 결과 반영", "코드 개선해줘", "지적 사항 반영", "코드 품질 개선", "코드 리뷰 반영해줘", "이슈 수정해줘" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC 구현 코드 수정 (코드 리뷰 반영)

`ai-dlc-sb-code-review` 검토 보고서의 VI-NNN 이슈를 기반으로 **Controller/Service/Mapper/VO 소스코드**를 실제 파일에 반영한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "코드 수정해줘", "리뷰 결과 반영", "코드 개선해줘"
- "지적 사항 반영", "코드 품질 개선", "코드 리뷰 반영해줘"

## 입력

- **필수**: 검토 보고서 (`ai-dlc-sb-code-review` 산출물) 또는 이슈 설명
- **필수**: 대상 소스코드 파일 경로

## 수정 절차

1. **이슈 목록 파악**: VI-NNN 이슈 코드·파일명·라인·수정 방안 읽기
2. **이슈 유형별 수정 패턴**:

   | 이슈 코드 | 수정 패턴 |
   |:---|:---|
   | SC-001 (SQL Injection) | MyBatis XML `${}` → `#{}` 전면 교체 |
   | SC-003 (인증 누락) | `@PreAuthorize("hasRole('USER')")` 추가 |
   | SC-004 (민감 정보 노출) | 로그·응답에서 비밀번호·토큰 필드 제거 |
   | LC-004 (@Autowired) | 필드 주입 제거, `final` + `@RequiredArgsConstructor` 변환 |
   | LC-001 (Controller 로직) | 비즈니스 코드 Service로 이동 |
   | PF-001 (N+1) | 반복 조회 → JOIN/IN 쿼리로 Mapper XML 수정 |
   | PF-002 (페이징 미적용) | LIMIT/OFFSET 파라미터 추가 |
   | CC-003 (메서드 길이) | Private 메서드 분리 |

3. **코드 위치 특정**: Glob/Read로 대상 파일 확인
4. **Edit 도구로 수정**: 최소 변경 원칙
5. **수정 완료 요약**: 대화창에 VI-NNN → 적용 결과 표 출력

## 수정 원칙

- **최소 변경**: 이슈에 지목된 코드만 수정, 불필요한 리팩터링 금지
- **보안 우선**: Critical 이슈(보안 취약점)를 먼저 처리
- **컴파일 안전**: import 추가·삭제, 타입 변경 시 컴파일 오류 유발하지 않도록 확인
- **로직 보존**: 기능적 동작을 변경하지 않고 품질 기준만 개선

## 산출물

- 수정된 소스코드 파일 (Controller/Service/Mapper/VO)
- 수정 완료 요약 (대화창 출력): VI-NNN → 수정 결과 표
