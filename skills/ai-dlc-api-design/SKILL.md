---
name: ai-dlc-api-design
description: AI-DLC 설계단계 스킬. 유즈케이스·화면 정의서 기반으로 OpenAPI 3.0 API 설계서를 생성한다. "API 설계서 만들어줘", "API 설계해줘", "OpenAPI 설계 문서 생성", "REST API 설계", "API 명세서 작성", "엔드포인트 설계", "API 스펙 설계해줘", "REST 설계 문서" 같은 표현이 나오면 반드시 이 스킬을 사용하라. 소스코드가 아닌 요구사항·UC 기반 신규 설계일 때 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC OpenAPI 3.0 API 설계서 생성

유즈케이스·화면 정의서·서비스 카탈로그를 기반으로 요구사항 순공학 방식의 **OpenAPI 3.0 YAML** 설계서와 MD 요약을 동시 생성한다.

> `ai-dlc-api-spec-extract`(소스코드 역공학)와 구분: 이 스킬은 **요구사항/UC 기반 신규 설계** 전용이다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "API 설계서 만들어줘", "API 설계해줘", "OpenAPI 설계 문서 생성", "REST API 설계"
- "API 명세서 작성", "엔드포인트 설계", "API 스펙 설계해줘", "REST 설계 문서"
- UC 문서·화면 정의서를 주며 "API로 변환해줘"라고 할 때

## 입력

- **필수**: 아래 중 1개 이상
  - 유즈케이스 문서 (`ai-dlc-usecase-create` 산출물)
  - 화면 정의서 (`ai-dlc-screen-spec` 산출물, 이벤트·API 매핑 정보)
- **선택**: 서비스 카탈로그(SC), `api-spec-extract` 산출물 (참조용), 기본 URL, 보안 스킴 지정

## 분석 절차

1. **API 후보 도출**: UC 흐름·화면 이벤트에서 API 후보 추출
   - UC 기본 흐름의 시스템 단계 → API 후보
   - 화면 이벤트(버튼클릭, 페이지로드, 조건변경) → API 후보
2. **RESTful 네이밍 적용**:
   - 리소스 명사형 경로: `/users`, `/orders/{orderId}`
   - 올바른 HTTP 메서드: GET(조회), POST(생성), PUT/PATCH(수정), DELETE(삭제)
   - 컬렉션/단건 구분: `/users`(목록), `/users/{id}`(단건)
3. **요청/응답 스키마 설계**: components/schemas에 DTO 정의
4. **보안 스킴 결정**: JWT Bearer, API Key, OAuth2 중 적합한 방식
5. **공통 응답 코드 정의**: 200, 201, 400, 401, 403, 404, 500
6. **OpenAPI 3.0 YAML 생성**: `API설계서_{사업명}_{YYYYMMDD}.yaml`
7. **MD 요약 동시 생성**: `API설계서_{사업명}_{YYYYMMDD}.md`

## 산출물 포맷

### YAML 파일 골격 (`API설계서_{사업명}_{YYYYMMDD}.yaml`)
```yaml
openapi: 3.0.3
info:
  title: {사업명} API
  version: 0.1.0
  description: 유즈케이스 기반 API 설계서 초안
servers:
  - url: {기본URL}
    description: 개발 서버
paths:
  /{리소스}:
    get: ...
    post: ...
components:
  schemas: ...
  securitySchemes: ...
```

미확인 항목: `# TODO: 확인 필요` 주석 필수

### MD 요약 파일 (`${CLAUDE_SKILL_DIR}/template.md` 사용)
1. **API 설계 개요** (엔드포인트 수, 리소스 목록, 보안 스킴)
2. **엔드포인트 목록** (메서드·경로·설명·UC 연계)
3. **보안 스킴 정의**
4. **공통 응답 코드**

## 작성 원칙

- 동사형 경로(`/getUser`, `/createOrder`) 금지 — 명사형 리소스 사용
- 스키마 미확인 항목은 `# TODO: 확인 필요` 주석 필수
- YAML + MD 동시 저장 (MD는 팀 공유용 요약)

## 엣지 케이스

- **GraphQL 요청**: REST 외에 GraphQL 명세도 별도 섹션으로 작성
- **파일 업로드 포함**: `multipart/form-data` 스키마 포함
- **페이지네이션**: Cursor/Offset 방식 명시, 공통 페이지네이션 스키마 정의
