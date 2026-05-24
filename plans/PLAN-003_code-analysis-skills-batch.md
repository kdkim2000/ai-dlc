# PLAN-003: AI-DLC 코드분석단계 스킬 8종 일괄 생성

## 개요

| 항목 | 내용 |
|:---|:---|
| 계획 번호 | PLAN-003 |
| 작성일 | 2026-05-23 |
| 상태 | 완료 |
| 선행 계획 | PLAN-001, PLAN-002 |

## 배경 및 목적

PLAN-001(요구사항 정의서)·PLAN-002(분석단계 10종)가 완료되어 요구사항 수립 단계 스킬이 갖춰졌다. 코드분석단계는 기존 소스코드를 읽어 영향도·의존성·복잡도를 파악하고 분석 결과를 표준 문서로 산출하는 단계로, PLAN-002의 요구사항 산출물(FR/BR)과 실제 소스코드를 연결하는 가교 역할을 한다.

## 스킬 목록 (8종)

### 사용자 지정 (3종)

| # | 스킬명 | 역할 | 산출물 파일명 |
|:---:|:---|:---|:---|
| 1 | `ai-dlc-impact-analysis` | 소스코드 기반 영향도 분석 | `영향도분석_{대상명}_{YYYYMMDD}.md` |
| 2 | `ai-dlc-md-to-word` | 마크다운 → 워드 문서 변환 | `{파일명}.docx` |
| 3 | `ai-dlc-program-spec` | 프로그램 분석서 생성 | `프로그램분석서_{시스템명}_{YYYYMMDD}.md` |

### 추가 제안 (5종)

| # | 스킬명 | 역할 | 산출물 파일명 |
|:---:|:---|:---|:---|
| 4 | `ai-dlc-code-traceability` | 코드-요구사항 추적성 매트릭스 | `추적성매트릭스_{시스템명}_{YYYYMMDD}.md` |
| 5 | `ai-dlc-dependency-analysis` | 코드 의존성 분석 | `의존성분석_{시스템명}_{YYYYMMDD}.md` |
| 6 | `ai-dlc-code-complexity` | 코드 복잡도·품질 분석 | `복잡도분석_{시스템명}_{YYYYMMDD}.md` |
| 7 | `ai-dlc-api-spec-extract` | API 명세 추출 (OpenAPI YAML) | `openapi_{시스템명}_{YYYYMMDD}.yaml` |
| 8 | `ai-dlc-data-model-analysis` | 데이터 모델 분석 + ERD | `데이터모델분석_{시스템명}_{YYYYMMDD}.md` |

## 생성 파일 목록

```
C:\Users\kdkim2000\.claude\skills\
├── ai-dlc-impact-analysis\
│   ├── SKILL.md       ✅
│   └── template.md    ✅
├── ai-dlc-md-to-word\
│   └── SKILL.md       ✅
├── ai-dlc-program-spec\
│   ├── SKILL.md       ✅
│   └── template.md    ✅
├── ai-dlc-code-traceability\
│   ├── SKILL.md       ✅
│   └── template.md    ✅
├── ai-dlc-dependency-analysis\
│   ├── SKILL.md       ✅
│   └── template.md    ✅
├── ai-dlc-code-complexity\
│   ├── SKILL.md       ✅
│   └── template.md    ✅
├── ai-dlc-api-spec-extract\
│   └── SKILL.md       ✅
└── ai-dlc-data-model-analysis\
    ├── SKILL.md       ✅
    └── template.md    ✅
```

총 생성 파일: **14개** (SKILL.md 8 + template.md 6)

## 공통 설계 원칙

1. **소스코드 읽기 도구**: `allowed-tools: Read Grep Glob` — 모든 코드분석 스킬 공통
2. **md-to-word 예외**: `allowed-tools: Bash(pandoc *) Bash(python *) Bash(pip *) Bash(where *) Bash(which *) Read`
3. **언어 자동 감지**: 파일 확장자(`.py`, `.ts`, `.java`, `.go` 등) 기반
4. **경로 입력**: 분석 대상 경로를 인자로 받거나 자연어로 설명 가능
5. **파일명 규칙**: `{산출물유형}_{대상시스템명}_{YYYYMMDD}.md`
6. **대용량 코드**: 파일 수 100개 초과 시 핵심 파일 자동 우선순위화
7. **보안 주의**: 시크릿 하드코딩 발견 시 분석서에 보안 경고 표시

## 트리거 키워드

| 스킬명 | 주요 트리거 |
|:---|:---|
| `ai-dlc-impact-analysis` | "영향도 분석", "변경하면 뭐가 바뀌어?", "어디까지 영향 가?" |
| `ai-dlc-md-to-word` | "워드로 변환", "docx로 바꿔줘", "마크다운을 워드로" |
| `ai-dlc-program-spec` | "프로그램 분석서", "코드 분석서 만들어줘", "소스코드 문서화" |
| `ai-dlc-code-traceability` | "코드 추적성", "FR 코드 매핑", "traceability matrix" |
| `ai-dlc-dependency-analysis` | "의존성 분석", "순환 의존", "어떤 패키지 쓰는지" |
| `ai-dlc-code-complexity` | "코드 복잡도", "리팩토링 후보", "사이클로매틱 복잡도" |
| `ai-dlc-api-spec-extract` | "API 명세 추출", "OpenAPI 생성", "Swagger 만들어줘" |
| `ai-dlc-data-model-analysis` | "데이터 모델 분석", "ERD 만들어줘", "DB 스키마 분석" |

## 비범위

- 실시간 코드 실행·빌드
- SonarQube·ESLint 등 외부 품질 도구와의 직접 연동
- 코드 자동 수정·리팩토링 (분석 및 제안만 수행)
- Git 이력 분석 (blame, log 기반 변경 이력)
- 성능 프로파일링

## 문서 버전 이력

| 버전 | 일자 | 내용 |
|:---|:---|:---|
| v1.0 | 2026-05-23 | PLAN-003 수립 및 실행 완료 |
