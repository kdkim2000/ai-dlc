---
name: ai-dlc-doc-impact
description: AI-DLC 변경 관리 스킬. 코드 변경 대상을 받아 수정이 필요한 설계 산출물을 자동 판단하고 권장 revise 스킬 체인을 출력한다. "설계 문서 영향 분석", "어떤 설계서 바꿔야 해", "문서 레벨 영향도", "산출물 영향 분석", "어떤 산출물이 영향받아" 같은 표현이 나오면 반드시 이 스킬을 사용하라. ai-dlc-impact-analysis(코드→코드)의 보완 스킬로 코드→설계 문서 역추적을 담당한다.
allowed-tools: Read Grep Glob
---

# AI-DLC 문서 레벨 영향도 분석

> `ai-dlc-impact-analysis`가 코드→코드 영향을 분석하는 것과 달리,  
> 이 스킬은 **코드→설계 문서** 역추적을 담당한다.  
> "이 코드를 바꾸면 어떤 설계 산출물을 revise해야 하는가?"에 답한다.

## 트리거

- "설계 문서 영향 분석", "어떤 설계서 바꿔야 해"
- "문서 레벨 영향도", "산출물 영향 분석"
- "어떤 산출물이 영향받아", "문서 영향도 확인"
- `ai-dlc-impact-analysis` 결과를 주며 "설계서 영향도 분석해줘"라고 할 때

## 입력

### 필수
- 변경 대상 코드 파일 목록 또는 `ai-dlc-impact-analysis` 결과 텍스트

### 선택
- `설계산출물/` 경로 (기본값: `./설계산출물/`)
- CR-NNN (해당 변경의 컨텍스트 제공용)

## 처리 절차

1. **현행 설계 산출물 목록 파악**
   - `설계산출물/` 디렉터리 스캔
   - 동일 유형의 파일이 여러 날짜 버전인 경우 → 날짜가 가장 최신인 파일이 현행 버전

2. **코드 파일 → 설계 ID 역추적**
   아래 매핑 규칙 적용:

   | 코드 파일 패턴 | 설계 ID | 관련 산출물 |
   |:---|:---|:---|
   | `src/app/(main)/[domain]/page.tsx` | SCR-NNN | 화면정의서, 화면목록 |
   | `src/app/(main)/[domain]/*/page.tsx` | SCR-NNN | 화면정의서 |
   | `src/app/api/[endpoint]/route.ts` | operationId | API설계서 |
   | `src/db/schema.ts` (테이블명) | 엔터티명 | 데이터설계서 |
   | `src/actions/[domain].ts` | UC-NNN | 유즈케이스 |
   | `src/components/[Domain]*.tsx` | SCR-NNN | 화면정의서 |
   | `src/lib/utils.ts` (비즈니스 함수) | FR-NNN | 비즈니스 규칙서, 유즈케이스 |

3. **설계 산출물별 revise 필요 여부 판단**
   - **revise 필요**: 설계 스펙 자체가 변경된 경우 (API 응답 필드 추가, DB 컬럼 추가, 화면 구성 변경, 비즈니스 로직 변경)
   - **revise 불필요**: 구현만 변경된 경우 (버그 수정, 성능 최적화, 리팩토링)
   - 판단 기준: 설계서를 읽고 구현한 사람이 "설계서대로 만들었는데 틀림"인가? → revise 불필요. "설계서가 틀림"인가? → revise 필요

4. **권장 revise 스킬 체인 생성**
   수정 필요 산출물에 대해 실행 순서 결정:
   - 상위 산출물 먼저 (UC → SCR → API/DATA → 화면정의서 순)
   - 각 revise 후 validate 쌍으로 구성

5. **결과 출력** (터미널 출력, 파일 저장 불필요)

## 출력 포맷

```markdown
## 문서 레벨 영향도 분석 결과

**분석 대상**: [변경 파일 목록 요약]
**분석 일시**: YYYY-MM-DD

---

### 수정 필요 산출물

| 산출물 유형 | 현행 파일명 | 역추적 근거 | revise 이유 | 권장 스킬 |
|:---|:---|:---|:---|:---|
| 데이터설계서 | 데이터설계서_CMMC_20260523.md | schema.ts:checklistItems | weight 컬럼 값 변경 | /ai-dlc-data-revise |
| 화면정의서 | 화면정의서_CMMC_20260523.md | SCR-004 (assessment/level2) | 데이터 표시 항목 변경 | /ai-dlc-screen-revise |

### 수정 불필요 산출물

| 산출물 유형 | 이유 |
|:---|:---|
| 유즈케이스 | UC 흐름 변경 없음 — 구현 오류 수정만 |
| API설계서 | 엔드포인트·스키마 스펙 변경 없음 |

---

### 권장 실행 순서

```
Step 1: /ai-dlc-data-revise @설계산출물/데이터설계서_CMMC_*.md [변경 내용]
Step 2: /ai-dlc-data-validate @설계산출물/데이터설계서_CMMC_*.md
Step 3: /ai-dlc-screen-revise @설계산출물/화면정의서_CMMC_*.md [변경 내용]
Step 4: /ai-dlc-screen-validate @설계산출물/화면정의서_CMMC_*.md
Step 5: 코드 반영 (/ai-dlc-nxt-code-revise 또는 직접 수정)
Step 6: /ai-dlc-nxt-code-review
Step 7: /ai-dlc-change-complete CR-NNN
```

---

### 일관성 검증 권장

여러 산출물을 revise한 경우 최종 검증 권장:
```
/ai-dlc-consistency-check @설계산출물/
```
```

## 엣지 케이스

- **설계산출물/ 디렉터리 없음**: "설계산출물 디렉터리를 찾을 수 없습니다" 메시지 출력 + 경로 입력 요청
- **코드 파일이 매핑 규칙에 없음**: "매핑 규칙 외 파일" 섹션에 별도 나열, 수동 판단 안내
- **설계서가 없는 도메인**: 신규 생성 필요 여부 안내 (CR-NEW인 경우 /ai-dlc-xxx-create 스킬 실행 권장)
- **impact-analysis 결과 없이 파일 목록만 제공**: 파일 목록 기준으로 직접 분석 수행
