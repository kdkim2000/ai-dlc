
# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 프로젝트 개요

이 워크스페이스는 **AI-DLC(AI-Driven Development Lifecycle)** 방법론을 지원하는 Claude Code **스킬(skill) 개발 프로젝트**다. 각 AI-DLC 단계(요구사항 정의 → 분석 → 설계 → 구현 → 검증)에 필요한 스킬을 계획·생성·관리한다.

## 스킬 위치

모든 스킬 파일은 `C:\Users\kdkim2000\.claude\skills\` 하위에 생성한다. 기존 `my-skills` 플러그인 캐시(`plugins\cache\`)는 플러그인 업데이트 시 덮어쓰일 수 있으므로 사용하지 않는다.

## 계획(Plan) 관리 규칙

Plan 모드에서 스킬 생성 계획을 수립하고 실행한 경우, **반드시** 아래 절차에 따라 `plans/` 폴더에 이력을 기록한다.

### 파일 명명 규칙

```
plans/PLAN-{NNN}_{주제_식별자}.md
```

- `NNN`: 3자리 순번. `plans/README.md`의 계획 목록에서 현재 마지막 번호 +1
- `주제_식별자`: kebab-case 영문 요약 (예: `ai-dlc-requirements-skill`, `analysis-skills-batch`)

### 계획 파일에 포함할 내용

각 `PLAN-NNN_*.md` 파일에는 다음을 누락 없이 기록한다:

1. **메타 표** — 작성일, 상태(진행중/완료), 전제 플랜, 스킬 경로
2. **Context** — 이 계획이 필요한 배경과 목적
3. **사용자 요구사항** — 사용자가 요청한 내용 원문 또는 요약
4. **설계 결정 사항** — 설계 시 내린 주요 결정과 근거 (표 형식 권장)
5. **생성된 파일 구조** — 실제 생성된 디렉터리·파일 트리
6. **스킬별 핵심 설계** — 각 스킬의 트리거·핵심 로직·포맷 요약
7. **검증 방법** — 올바르게 동작하는지 확인하는 테스트 시나리오
8. **비범위** — 이번 계획에서 의도적으로 제외한 항목

### `plans/README.md` 업데이트

계획 파일 생성 후 `plans/README.md`의 **계획 목록 표**에 새 행을 추가한다:

```markdown
| {NNN} | [PLAN-{NNN}_{식별자}.md](PLAN-{NNN}_{식별자}.md) | {제목} | {YYYY-MM-DD} | 완료 |
```

산출물(스킬)이 새로 생성된 경우 **산출물 위치** 표도 함께 업데이트한다.

### 실행 순서 요약

```
Plan 모드 수립 → 사용자 승인 → 스킬 파일 생성
→ plans/PLAN-NNN_*.md 작성 → plans/README.md 업데이트
```

## 기존 스킬 목록

현재까지 생성된 스킬은 `plans/README.md`의 산출물 위치 표를 참조한다.
