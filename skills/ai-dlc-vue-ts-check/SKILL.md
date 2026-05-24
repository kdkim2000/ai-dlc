---
name: ai-dlc-vue-ts-check
description: AI-DLC 개발단계(프론트엔드-Vue.js) 스킬. vue-tsc TypeScript 검사 결과를 분석하여 정리된 보고서를 생성한다. "Vue TypeScript 검사해줘", "vue-tsc 실행해줘", "SFC 타입 오류 정리", "Vue 타입 검사 결과 분석", "vue-tsc 오류 분류" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Vue.js TypeScript 검사 (vue-tsc)

`vue-tsc --noEmit` 실행 결과를 분석하여 SFC 타입 오류를 유형별로 분류하고 수정 우선순위를 제시하는 보고서를 생성한다.
공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "Vue TypeScript 검사해줘", "vue-tsc 실행해줘", "SFC 타입 오류 정리"
- "Vue 타입 검사 결과 분석", "vue-tsc 오류 분류"
- "TypeScript 타입 오류 정리해줘 (Vue 프로젝트)"

---

## 입력

### 필수
- `vue-tsc --noEmit` 실행 결과 (오류 메시지 붙여넣기 또는 파일 경로)

### 선택
- 프로젝트 소스 경로 (오류 맥락 파악 시 읽기)
- `tsconfig.json` 경로 (strict 설정 확인)

---

## 분석 절차

### 1단계: 오류 목록 파싱
vue-tsc 출력에서 오류 항목을 파일·라인·오류 코드(TS####)·메시지로 파싱.

### 2단계: 오류 유형 분류

| 유형 | 설명 | 예시 오류 코드 |
|:---|:---|:---|
| **SFC Props 타입 불일치** | 부모가 전달한 prop 타입과 `defineProps<T>` 불일치 | TS2322 |
| **Composable 반환 타입 누락** | `useXxx()` 반환값 타입 추론 실패 | TS2345, TS7006 |
| **ref/reactive 제네릭 미지정** | `ref()`, `reactive()` 타입 파라미터 없음 | TS7005 |
| **undefined 가능성** | `?.` 없이 nullable 접근 | TS2532, TS18047 |
| **any 타입 사용** | 명시적 또는 암묵적 any | TS7006, TS2742 |
| **미사용 변수/import** | `noUnusedLocals`, `noUnusedParameters` 위반 | TS6133, TS6196 |
| **기타 타입 오류** | 위 분류에 포함되지 않는 오류 | - |

### 3단계: 심각도 분류
- **높음**: SFC Props 타입 불일치, any 타입, undefined 가능성 (런타임 오류 유발 가능)
- **중간**: Composable 반환 타입 누락, ref 제네릭 미지정 (타입 안전성 약화)
- **낮음**: 미사용 변수/import (코드 품질)

### 4단계: 수정 우선순위 제안
높음 → 중간 → 낮음 순서로 수정 제안 작성.

---

## 산출물

`TypeScript검사결과_{사업명}_{YYYYMMDD}.md`

### 보고서 구성

```markdown
# Vue.js TypeScript 검사 결과 (vue-tsc)

| 항목 | 내용 |
|:---|:---|
| 검사 일자 | YYYY-MM-DD |
| 검사 명령 | `vue-tsc --noEmit` |
| 총 오류 수 | N건 |
| TypeScript 버전 | 5.x |

## 오류 요약

| 파일 | 라인 | 오류 코드 | 유형 | 심각도 | 메시지 요약 |
|:---|:---:|:---:|:---|:---:|:---|
| UserForm.vue | 12 | TS2322 | Props 타입 불일치 | 높음 | ... |

## 오류 유형별 통계

| 유형 | 건수 |
|:---|:---:|
| SFC Props 타입 불일치 | N |
| Composable 반환 타입 누락 | N |
| ref/reactive 제네릭 미지정 | N |
| undefined 가능성 | N |
| any 타입 사용 | N |
| 미사용 변수/import | N |

## 수정 우선순위

### 높음 (즉시 수정)
1. `UserForm.vue:12` — Props 타입 불일치: ...

### 중간 (다음 스프린트 내)
...

### 낮음 (리팩터링 시)
...
```

## 문서 버전 이력

| 버전 | 일자 | 작성자 | 변경 내용 |
|:---|:---|:---|:---|
| v0.1 | YYYY-MM-DD | 초안 작성 | 최초 생성 |
```
