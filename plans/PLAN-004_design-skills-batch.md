# PLAN-004: AI-DLC 설계단계 스킬 18종 일괄 생성

## 메타

| 항목 | 내용 |
|:---|:---|
| 작성일 | 2026-05-23 |
| 상태 | 완료 |
| 전제 플랜 | PLAN-001, PLAN-002, PLAN-003 |
| 스킬 경로 | `C:\Users\kdkim2000\.claude\skills\` |

---

## Context

PLAN-001~003이 완료되어 요구사항 정의·분석·코드분석 단계 스킬(총 20종)이 갖춰졌다. 설계단계(Design Phase)에 필요한 스킬을 일괄 생성한다. 설계단계는 분석단계 산출물(FR/BR/UC 후보)을 구체적인 설계 문서(유즈케이스·화면·API·데이터·클래스)로 변환하는 단계다.

---

## 사용자 요구사항

```
AI-DLC 설계단계에서 필요한 skills 의 목록을 도출하고 일괄 생성하는 계획을 작성한다.
설계단계에서는 아래 skills가 포함될 수 있도록 한다.
- 기존 API/DDL 산출물에서 설계 추출 문서 생성
- 서비스 요구사항 기반 유즈케이스 시나리오 성성
- 요구사항-유즈케이스 일관성 검증
- 유즈케이스 검증 결과 반영
- 메뉴 구조 및 화면 목록 도출
- 화면 정의서 생성
- 유즈케이스-화면 문서 일관성 검증
- 화면 검증 결과 반영
- OpenAPI 3.0 기반 API 설계서 생성
- API 설계 문서 검증
- API 검증 결과 반영
- 데이터 설계서 생성
- 데이터 설계 문서 검증
- 클래스 설계서 생성
그외 설계단계에서 필요한 skills 중 누락된 부분이 있다면 추가하여 계획을 수립하라.
```

---

## 설계 결정 사항

| 결정 | 내용 | 근거 |
|:---|:---|:---|
| 추가 4종 | data-revise, class-validate, class-revise, sequence-design | create/validate/revise 3종 세트 완성 + 시퀀스는 독립 핵심 산출물 |
| api-design vs api-spec-extract 구분 | api-design=순공학(UC→API), api-spec-extract=역공학(코드→API) | 사용 맥락 완전히 다름, SKILL.md에 구분 명시 |
| YAML + MD 이중 출력 | api-design만 OpenAPI YAML + MD 요약 동시 생성 | API는 YAML이 표준, MD는 팀 공유용 |
| Mermaid 도구 | classDiagram/erDiagram/sequenceDiagram | 외부 도구 불필요, 텍스트로 버전 관리 가능 |
| allowed-tools | 전 스킬 `Read Grep Glob` | 설계 산출물 읽기 전용, 소스코드 직접 수정 금지 |
| ASCII 레이아웃 | 화면 정의서에 텍스트 박스로 레이아웃 표현 | 이미지 생성은 비범위 |

---

## 생성된 파일 구조

```
C:\Users\kdkim2000\.claude\skills\
├── ai-dlc-design-extract\
│   ├── SKILL.md
│   └── template.md
├── ai-dlc-usecase-create\
│   ├── SKILL.md
│   └── template.md
├── ai-dlc-usecase-validate\
│   └── SKILL.md
├── ai-dlc-usecase-revise\
│   └── SKILL.md
├── ai-dlc-screen-list\
│   ├── SKILL.md
│   └── template.md
├── ai-dlc-screen-spec\
│   ├── SKILL.md
│   └── template.md
├── ai-dlc-screen-validate\
│   └── SKILL.md
├── ai-dlc-screen-revise\
│   └── SKILL.md
├── ai-dlc-api-design\
│   ├── SKILL.md
│   └── template.md           ← MD 요약 템플릿 (YAML은 인라인 골격)
├── ai-dlc-api-validate\
│   └── SKILL.md
├── ai-dlc-api-revise\
│   └── SKILL.md
├── ai-dlc-data-design\
│   ├── SKILL.md
│   └── template.md
├── ai-dlc-data-validate\
│   └── SKILL.md
├── ai-dlc-data-revise\
│   └── SKILL.md
├── ai-dlc-class-design\
│   ├── SKILL.md
│   └── template.md
├── ai-dlc-class-validate\
│   └── SKILL.md
├── ai-dlc-class-revise\
│   └── SKILL.md
└── ai-dlc-sequence-design\
    ├── SKILL.md
    └── template.md
```

**총 파일: 26개** (SKILL.md 18 + template.md 8)

---

## 스킬별 핵심 설계

| 스킬명 | 트리거 (대표) | 핵심 로직 | 산출물 |
|:---|:---|:---|:---|
| ai-dlc-design-extract | "설계 기초 뽑아줘" | PLAN-003 산출물에서 UC/화면/클래스 후보 추출 | 설계기초추출서 |
| ai-dlc-usecase-create | "유즈케이스 작성" | FR → UC-NNN 채번, 기본/대안/예외 흐름, Mermaid UC 다이어그램 | 유즈케이스_{사업명}.md |
| ai-dlc-usecase-validate | "유즈케이스 검증" | 누락/불완전/모순/중복/형식오류 탐지, FR 커버리지 | 유즈케이스_검증.md |
| ai-dlc-usecase-revise | "유즈케이스 수정" | VI-NNN 반영 또는 자연어, 버전 +0.1 | 유즈케이스_{사업명}_v{N.N}.md |
| ai-dlc-screen-list | "화면 목록 도출" | UC/FR → SCR-NNN, GNB>LNB>화면 3단계, 역할별 접근 | 화면목록_{사업명}.md |
| ai-dlc-screen-spec | "화면 정의서 만들어줘" | SCR별 레이아웃·I/O·이벤트·API 연계 | 화면정의서_{사업명}.md |
| ai-dlc-screen-validate | "화면 설계 검증" | UC 커버리지·API 불일치·UX 결함·형식오류 | 화면설계_검증.md |
| ai-dlc-screen-revise | "화면 설계 수정" | VI-NNN 반영, SCR-NNN +1 채번, 흐름도 동시 갱신 | 화면정의서_{사업명}_v{N.N}.md |
| ai-dlc-api-design | "API 설계서 만들어줘" | UC/화면 → RESTful 엔드포인트, OpenAPI YAML + MD | API설계서_{사업명}.yaml + .md |
| ai-dlc-api-validate | "API 설계 검증" | UC 커버리지·RESTful 위반·스키마·보안 누락 | API설계_검증.md |
| ai-dlc-api-revise | "API 설계 수정" | VI-NNN 반영, YAML+MD 동시 수정, info.version +0.1 | API설계서_{사업명}_v{N.N}.yaml + .md |
| ai-dlc-data-design | "데이터 설계서 만들어줘" | UC/FR → 엔터티 → 논리/물리 ERD + DDL | 데이터설계서_{사업명}.md |
| ai-dlc-data-validate | "데이터 설계 검증" | 정규화(3NF)·참조 무결성·API 불일치·명명 규칙 | 데이터설계_검증.md |
| ai-dlc-data-revise | "데이터 설계 수정" | VI-NNN 반영, ERD+테이블+DDL 동시 수정 | 데이터설계서_{사업명}_v{N.N}.md |
| ai-dlc-class-design | "클래스 설계서 만들어줘" | UC/API → CLS-NNN, 4계층, Mermaid classDiagram | 클래스설계서_{사업명}.md |
| ai-dlc-class-validate | "클래스 설계 검증" | 순환의존·SRP 위반·레이어 경계·UC 커버리지 | 클래스설계_검증.md |
| ai-dlc-class-revise | "클래스 설계 수정" | VI-NNN 반영, classDiagram+상세 테이블 동시 수정 | 클래스설계서_{사업명}_v{N.N}.md |
| ai-dlc-sequence-design | "시퀀스 다이어그램 만들어줘" | UC 흐름 → SEQ-NNN, Mermaid sequenceDiagram | 시퀀스다이어그램_{사업명}.md |

---

## ID 채번 체계

| ID | 산출물 | 채번 방식 |
|:---|:---|:---|
| UC-NNN | 유즈케이스 | 001부터 순차 |
| SCR-NNN | 화면 | 001부터 순차 |
| CLS-NNN | 클래스 | 001부터 순차 |
| SEQ-NNN | 시퀀스 다이어그램 | 001부터 순차 |
| VI-NNN | 검증 이슈 | validate 스킬 공통 |

---

## 검증 방법

| 트리거 문장 | 기대 스킬 |
|:---|:---|
| "유즈케이스 시나리오 만들어줘" | `ai-dlc-usecase-create` |
| "화면 목록 뽑아줘" | `ai-dlc-screen-list` |
| "화면 정의서 만들어줘" | `ai-dlc-screen-spec` |
| "API 설계서 만들어줘" | `ai-dlc-api-design` |
| "API 스펙 뽑아줘" | `ai-dlc-api-spec-extract` (PLAN-003, 역공학) |
| "ERD 설계해줘" | `ai-dlc-data-design` |
| "클래스 다이어그램 만들어줘" | `ai-dlc-class-design` |
| "시퀀스 다이어그램 만들어줘" | `ai-dlc-sequence-design` |
| "UC 검증해줘" | `ai-dlc-usecase-validate` |
| "클래스 설계 SOLID 원칙 확인" | `ai-dlc-class-validate` |
| "데이터 설계 수정해줘" | `ai-dlc-data-revise` |

---

## 비범위

- 와이어프레임 이미지 생성 (텍스트 레이아웃 설명만)
- 코드 자동 생성·구현 (설계 문서만 생성)
- DB 스키마 실제 생성·마이그레이션 실행
- UML 도구(PlantUML 서버, draw.io)와의 직접 연동
- 성능 설계·용량 산정
