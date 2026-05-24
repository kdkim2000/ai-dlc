---
name: ai-dlc-api-revise
description: AI-DLC 설계단계 스킬. API 설계서를 수정한다(검증 결과 반영 또는 자연어 지시 기반). "API 설계 수정", "API 설계서 업데이트", "API 검증 반영", "OpenAPI 수정해줘", "엔드포인트 변경", "API 버전 올려줘", "API 스펙 수정" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC API 설계 수정

검증 보고서 또는 자연어 지시에 따라 API 설계서(YAML + MD 요약)를 수정하고 `info.version`을 +0.1 증가시켜 새 파일로 저장한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "API 설계 수정", "API 설계서 업데이트", "API 검증 반영", "OpenAPI 수정해줘"
- "엔드포인트 변경", "API 버전 올려줘", "API 스펙 수정"
- 검증 보고서를 주며 "반영해줘"라고 할 때
- API 설계서를 주며 "이 엔드포인트 고쳐줘"라고 할 때

## 입력

- **필수**: 기존 API 설계서 (`ai-dlc-api-design` 산출물: YAML 또는 MD 요약)
- **선택**: 검증 보고서 (`ai-dlc-api-validate` 산출물), 자연어 수정 지시

## 처리 절차

1. **입력 방식 판별**:
   - 검증 보고서 있음 → VI-NNN 이슈 목록에서 변경 사항 자동 추출
   - 검증 보고서 없음 → 자연어 지시에서 변경 사항 파싱
2. **변경 목록 정리**: 엔드포인트 추가/삭제/수정, 스키마 변경, 보안 스킴 변경
3. **불명확 지시 확인**: 1회만, 3개 이하로 묶어서 질문
4. **변경 적용**:
   - 엔드포인트 추가: RESTful 네이밍 준수, operationId 자동 생성
   - 엔드포인트 삭제: 해당 path 제거, components/schemas 연계 스키마 정리
   - 스키마 수정: `$ref` 참조 일관성 유지
   - 보안 수정: `securitySchemes` + 각 엔드포인트 `security` 동시 갱신
   - RESTful 위반 수정: 동사형 경로 → 명사형 리소스 변환
5. **버전 증가**: `info.version` +0.1, MD 요약 버전 이력 표 추가
6. **YAML + MD 동시 저장**: 두 파일 모두 새 버전으로 저장

## 변경 유형

| 변경 유형 | 처리 방법 |
|:---|:---|
| 엔드포인트 추가 | RESTful 명사형 경로, operationId 자동 생성, 스키마 동시 추가 |
| 엔드포인트 삭제 | path 제거, 미사용 스키마 정리 |
| 경로 수정 (RESTful 위반) | 동사형 → 명사형, operationId 재설정 |
| 스키마 필드 추가/수정 | components/schemas 수정, requestBody/responses 참조 일관성 유지 |
| 보안 스킴 변경 | securitySchemes 수정 + 영향 받는 모든 엔드포인트 security 동시 갱신 |
| TODO 해소 | `# TODO: 확인 필요` 주석 제거, 실제 스키마로 교체 |

## 파일명 규칙

- YAML: `API설계서_{사업명}_{YYYYMMDD}_v{N.N}.yaml`
- MD 요약: `API설계서_{사업명}_{YYYYMMDD}_v{N.N}.md`
- YAML과 MD 요약 버전 번호 동기화 필수

## 특이사항

- YAML과 MD 요약은 반드시 **동시 수정·저장**: 두 파일 간 불일치 금지
- `info.version` 변경 시 MD 요약 헤더의 버전 표기도 함께 갱신
- 기존 operationId 변경 시 화면 정의서·유즈케이스 문서의 연계 ID도 변경 필요 — 수정 완료 후 연계 문서 갱신 안내 메시지 출력
