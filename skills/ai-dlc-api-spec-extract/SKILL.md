---
name: ai-dlc-api-spec-extract
description: AI-DLC 코드분석단계 스킬. 소스코드 라우터/컨트롤러에서 REST API 엔드포인트를 추출해 OpenAPI YAML 초안을 생성한다. "API 명세 추출", "OpenAPI 생성", "Swagger 만들어줘", "API 문서화", "REST 명세", "API 스펙 뽑아줘", "엔드포인트 목록", "라우터 분석해줘" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC API 명세 추출

소스코드의 라우터·컨트롤러 파일을 분석하여 REST API 엔드포인트·파라미터·응답 코드를 추출하고 **OpenAPI 3.0.3 YAML 초안**을 생성한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "API 명세 추출", "OpenAPI 생성", "Swagger 만들어줘", "API 문서화"
- "REST 명세", "API 스펙 뽑아줘", "엔드포인트 목록", "라우터 분석해줘"
- "API 문서 자동 생성", "openapi yaml 만들어줘"
- 소스코드 경로를 주며 "API 명세 만들어줘"라고 할 때

## 입력

- **필수**: 분석 대상 경로 (디렉터리 또는 라우터/컨트롤러 파일)
- **선택**: 시스템명, API 기본 URL (`servers[].url`)

## 분석 절차

1. **라우터/컨트롤러 파일 탐지**: `Glob`으로 라우터·컨트롤러 파일 탐지
   - Express: `*router*`, `*routes*`, `*controller*`
   - FastAPI/Flask: `*router*`, `*api*`, `*views*`
   - Spring: `*Controller.java`, `*Resource.java`
   - NestJS: `*.controller.ts`
2. **엔드포인트 추출**: 언어별 패턴 `Grep`으로 탐색
   - 경로 (path), HTTP 메서드 (GET/POST/PUT/PATCH/DELETE)
   - 경로 파라미터 (`:id`, `{id}`, `<int:id>`)
   - 쿼리 파라미터 (함수 인자명 기반 추정)
3. **요청/응답 스키마 추출**:
   - TypeScript: DTO 클래스·인터페이스에서 필드 추출
   - Python: Pydantic 모델·dataclass에서 필드 추출
   - Java: DTO/Request 클래스에서 필드 추출
   - 불명확한 항목은 `# TODO: 확인 필요` 주석 표시
4. **보안 스킴 탐지**: JWT, API Key, OAuth2 관련 미들웨어·데코레이터 탐지
5. **OpenAPI YAML 초안 생성**: openapi 3.0.3 형식으로 직접 출력

## 산출물 포맷

파일명: `openapi_{시스템명}_{YYYYMMDD}.yaml`

```yaml
openapi: 3.0.3
info:
  title: {시스템명} API
  version: 0.1.0
  description: 소스코드에서 자동 추출한 API 명세 초안 (수동 검토 필요)
servers:
  - url: {기본URL}
paths:
  /{경로}:
    {method}:
      summary: {설명}
      parameters: [...]
      requestBody: {...}   # TODO: 스키마 확인 필요
      responses:
        '200':
          description: 성공
        '400':
          description: 잘못된 요청
components:
  schemas: {...}
  securitySchemes: {...}
```

미확인 항목에는 반드시 `# TODO: 확인 필요` 주석 표시.

## 언어/프레임워크별 탐지 패턴

| 프레임워크 | 탐지 패턴 |
|:---|:---|
| Express | `router.get(`, `router.post(`, `app.get(`, `router.delete(` |
| FastAPI | `@app.get(`, `@router.post(`, `@app.delete(` |
| Flask | `@app.route(`, `@blueprint.route(`, `methods=[` |
| Spring MVC | `@GetMapping`, `@PostMapping`, `@RequestMapping` |
| NestJS | `@Get(`, `@Post(`, `@Controller(` |
| Go (gin) | `r.GET(`, `r.POST(`, `r.DELETE(` |

## 작성 원칙

- `node_modules`, `.git`, `dist`, `build`, `__pycache__` 제외
- 스키마가 불명확한 경우 `# TODO: 확인 필요` 필수 표기
- 보안 스킴이 탐지된 경우 `security` 섹션 포함
- 엔드포인트 수가 50개 초과 시 태그별로 그룹화

## 엣지 케이스

- **동적 경로 생성**: 정적 분석 한계 명시, 탐지된 패턴만 포함
- **GraphQL**: REST 외에 GraphQL이 감지되면 별도 섹션에 스키마 요약
- **gRPC**: proto 파일 탐지 시 서비스·메서드 목록 별도 표
- **버전별 API**: `/v1/`, `/v2/` 경로가 있으면 버전별로 태그 분리
