# PLAN-011: AI-DLC Pre-Requirements 스킬 5종 일괄 생성

## 메타

| 항목 | 내용 |
|:---|:---|
| 작성일 | 2026-05-25 |
| 상태 | 완료 |
| 전제 플랜 | PLAN-001 (ai-dlc-requirements 기반) |
| 스킬 경로 | `C:\Users\kdkim2000\.claude\skills\` |

---

## Context

현재 AI-DLC 프로세스는 `ai-dlc-requirements`가 최상위 진입점이다.
그러나 "막연한 불편함·아이디어는 있지만 구체적 요구사항을 모를 때"의 진입점이 없었다.

사용자 요청:
> "요구사항이 명확한 경우는 ai-dlc-requirements로부터 시작하지만 불편하고 필요한 어플리케이션은 있으나 구체적인 요구사항이 없을 때 아이디어를 구체화하고 요구사항을 잘 뽑아 낼 수 있는 skills를 추가하고 싶다."

Pre-Requirements 스킬 5종을 신규 생성하여 **아이디어 → 요구사항** 경로를 완성한다.

---

## 사용자 요구사항

아이디어·불편함 → 구체화 → 요구사항 도출의 전 과정을 스킬 체인으로 지원:
1. 아이디어 구체화 (5W1H 인터뷰)
2. 사용자 페르소나 정의
3. 사용자 스토리 맵 작성
4. MVP 범위 정의 (MoSCoW)
5. 기존 AI-DLC(FR-NNN) 포맷으로 변환

---

## 설계 결정 사항

| 항목 | 결정 | 근거 |
|:---|:---|:---|
| 스킬 수 | 5종 (create-only) | 아이디어 단계 산출물은 탐색적 성격, validate/revise 불필요 |
| idea-to-req 포맷 | ai-dlc-requirements template.md와 동일 | 기존 파이프라인과 완전 호환 |
| US-NNN ID 도입 | 신규 (PS-NNN도 신규) | 페르소나·스토리 추적성 확보 |
| 인터뷰 방식 | 최대 5문항 1회 그룹 질문 | 기존 스킬 패턴 일관성 유지 |
| template.md | 전 스킬 포함 | 자리표시자 치환 패턴 일관성 |

---

## 생성된 파일 구조

```
C:\Users\kdkim2000\.claude\skills\
├── ai-dlc-idea-clarify\
│   ├── SKILL.md
│   └── template.md
├── ai-dlc-persona-create\
│   ├── SKILL.md
│   └── template.md
├── ai-dlc-user-story-map\
│   ├── SKILL.md
│   └── template.md
├── ai-dlc-mvp-scope\
│   ├── SKILL.md
│   └── template.md
└── ai-dlc-idea-to-req\
    ├── SKILL.md
    └── template.md

E:\apps\ai-dlc\skills\  (동일 구조, git 반영)
```

---

## 스킬별 핵심 설계

| 스킬 | 트리거 키워드 | 처리 핵심 | 산출물 | 다음 스킬 |
|:---|:---|:---|:---|:---|
| `ai-dlc-idea-clarify` | "아이디어 있는데", "앱 만들고 싶어", "이런 게 불편해" | 5W1H 분석 → 최대 5문항 그룹 인터뷰 | 아이디어정의서_*.md | persona-create, user-story-map |
| `ai-dlc-persona-create` | "페르소나 만들어줘", "사용자 정의해줘" | 아이디어정의서 탐색 → PS-NNN 카드 생성 | 페르소나_*.md | user-story-map |
| `ai-dlc-user-story-map` | "사용자 스토리 만들어줘", "US 만들어줘" | 여정 Activity → Task → US-NNN 변환 + MoSCoW | 사용자스토리맵_*.md | mvp-scope |
| `ai-dlc-mvp-scope` | "MVP 정의해줘", "MoSCoW" | MoSCoW + 가치/복잡도 매트릭스 + 릴리즈 로드맵 | MVP범위정의서_*.md | idea-to-req |
| `ai-dlc-idea-to-req` | "요구사항으로 변환해줘", "FR로 변환해줘" | US→FR 변환, ai-dlc-requirements template 재사용, US↔FR 매핑 | 요구사항정의서_*.md | service-catalog |

---

## 전체 흐름

```
[아이디어 / 불편함]
       │
       ▼
ai-dlc-idea-clarify (아이디어정의서)
       │
  ┌────┴────────┐
  ▼             ▼
ai-dlc-     ai-dlc-
persona-    user-story-
create      map
(PS-NNN)    (US-NNN)
  └────┬────┘
       ▼
ai-dlc-mvp-scope (MoSCoW)
       │
       ▼
ai-dlc-idea-to-req (FR-NNN 요구사항정의서)
       │
       ▼
[기존 AI-DLC: service-catalog → usecase-create → ...]
```

---

## 문서 업데이트 내역

| 파일 | 변경 내용 |
|:---|:---|
| `AI-DLC-README.md` | 2절 "아이디어 단계" 신규 섹션 추가, 목차 번호 재정렬, 스킬 전체 목록에 Pre-Requirements 5종 추가 |
| `README.md` | 스킬 수 110→115종, Pre-Requirements 5종 섹션 추가 |
| `plans/README.md` | PLAN-011 행 추가, 산출물 5종 추가 |

---

## 검증 방법

| 스킬 | 검증 시나리오 |
|:---|:---|
| `ai-dlc-idea-clarify` | "직원 근태관리가 불편한데 앱 만들고 싶어" → 아이디어정의서 생성, 5문항 인터뷰 동작 확인 |
| `ai-dlc-persona-create` | 아이디어정의서 파일 있을 때 PS-001, PS-002 자동 생성 확인 |
| `ai-dlc-user-story-map` | US-NNN 채번, Must/Should/Nice 분류, 수락 기준 포함 확인 |
| `ai-dlc-mvp-scope` | MoSCoW 표 + Quick Win 목록 + 릴리즈 로드맵 생성 확인 |
| `ai-dlc-idea-to-req` | FR-NNN 자동 채번, ai-dlc-requirements 포맷 동일성, US↔FR 매핑 테이블 확인 |
| 연계 검증 | idea-to-req 산출물로 ai-dlc-service-catalog 바로 실행 가능 여부 확인 |

---

## 비범위

- validate/revise 스킬 쌍: 아이디어 단계 산출물은 탐색적 성격으로 이번 배치 제외
- 경쟁사 분석, 시장 조사 스킬: 도메인 특화 내용으로 별도 계획 필요
- 기술 타당성 검토(PoC) 스킬: 개발 단계 스킬과 연계 필요, 별도 계획
