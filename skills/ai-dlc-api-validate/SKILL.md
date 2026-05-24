---
name: ai-dlc-api-validate
description: AI-DLC 설계단계 스킬. API 설계서의 완전성·RESTful 준수·스키마 일관성·보안 누락을 검증한다. "API 설계 검증", "API 설계서 검토", "OpenAPI 검증", "REST API 리뷰", "API 명세 검토해줘", "API 스펙 리뷰", "엔드포인트 검증" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC API 설계 검증

API 설계서(OpenAPI YAML 또는 MD 요약)를 검증하여 UC 커버리지 누락·RESTful 위반·스키마 불완전·보안 누락·형식오류를 탐지하고 **API 설계 검증 보고서**를 생성한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.
품질 기준: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/quality-checklist.md` API 설계 섹션 참조.

## 트리거

- "API 설계 검증", "API 설계서 검토", "OpenAPI 검증", "REST API 리뷰"
- "API 명세 검토해줘", "API 스펙 리뷰", "엔드포인트 검증"
- API 설계서를 주며 "검토해줘", "확인해줘"라고 할 때

## 입력

- **필수**: API 설계서 (`ai-dlc-api-design` 산출물: YAML 또는 MD 요약)
- **선택**: 유즈케이스 문서, 화면 정의서 (커버리지 검증용)

## 검증 항목 (5종)

| 유형 | 설명 | 심각도 |
|:---|:---|:---:|
| UC 커버리지 누락 | UC 기본 흐름의 시스템 단계가 API로 표현되지 않은 경우 | 높음 |
| RESTful 위반 | 동사형 경로(`/getUser`), 잘못된 HTTP 메서드(GET으로 생성 등) | 높음 |
| 스키마 불완전 | 요청/응답 스키마 미정의, 필수 필드 누락, `# TODO` 미해결 | 중간 |
| 보안 누락 | 인증이 필요한 엔드포인트에 `security` 미적용 | 중간 |
| 형식오류 | operationId 중복, 필수 필드(`summary`, `responses`) 누락 | 낮음 |

## 분석 절차

1. **API 설계서 파싱**: `paths`, `components/schemas`, `securitySchemes` 추출
2. **UC-API 커버리지 확인**: UC 문서가 있으면 UC 기본 흐름 단계 vs. operationId 교차 검증
3. **RESTful 준수 검사**:
   - 동사형 경로 탐지: `/get`, `/create`, `/delete`, `/update`, `/fetch`, `/add` 등 포함 경로
   - HTTP 메서드 의미론 검사: GET 요청에 request body, POST로 조회 처리 등
   - 컬렉션/단건 구분: `/{resource}` vs `/{resource}/{id}` 패턴
4. **스키마 완전성 검사**:
   - 모든 경로의 requestBody, responses 200/400/401 스키마 존재 여부
   - `# TODO` 주석이 있는 미완성 스키마 목록
   - `$ref` 참조 대상 스키마 존재 여부
5. **보안 적용 검사**: 공개 엔드포인트(로그인, 헬스체크 제외) `security` 키 누락 탐지
6. **형식오류 탐지**: operationId 중복, `summary` 누락, responses 비어있음
7. **VI-NNN 이슈 목록 작성**
8. **UC-API 커버리지 계산**: 커버된 UC 수 / 전체 UC 수 × 100
9. **종합 판정** 산출

## 판정 기준

| 판정 | 조건 |
|:---|:---|
| 승인 | 높음 이슈 0건, 전체 이슈 5건 이하, UC-API 커버리지 ≥ 90% |
| 조건부 승인 | 높음 이슈 0건, 전체 이슈 15건 이하, UC-API 커버리지 ≥ 75% |
| 재검토 필요 | 높음 이슈 1건 이상 또는 UC-API 커버리지 < 75% |

## 산출물 포맷

파일명: `API설계_검증_{YYYYMMDD}.md`

```markdown
# API 설계 검증 보고서

## 1. 검증 요약
| 항목 | 내용 |
|:---|:---|
| 검증 대상 엔드포인트 수 | N개 |
| 발견 이슈 수 | N건 (높음 N / 중간 N / 낮음 N) |
| UC-API 커버리지 | N% |
| 종합 판정 | 승인 / 조건부 승인 / 재검토 필요 |

## 2. 이슈 목록
| VI-ID | 경로/operationId | 이슈 유형 | 내용 | 심각도 | 권고 조치 |

## 3. RESTful 준수 현황
| 경로 | 메서드 | 현재 상태 | 권고 수정 |

## 4. UC-API 커버리지
| UC-ID | UC명 | 연계 operationId | 커버 상태 |
커버리지: N%

## 5. 미완성 스키마 목록 (TODO)
| 스키마명/경로 | TODO 내용 |

## 6. 종합 판정 및 의견
```

## 엣지 케이스

- **YAML 없이 MD 요약만 있는 경우**: MD 요약 기반으로 가능한 범위 내 검증, YAML 확보 권고
- **GraphQL 포함**: REST 외 GraphQL 엔드포인트는 별도 섹션으로 검증
- **외부 API 연동**: 서드파티 API 호출은 커버리지 산정에서 제외
