---
name: ai-dlc-vue-lint-check
description: AI-DLC 개발단계(프론트엔드-Vue.js) 스킬. eslint-plugin-vue 검사 결과를 분석하여 정리된 보고서를 생성한다. "Vue ESLint 검사해줘", "Vue lint 결과 정리", "eslint-plugin-vue 오류 정리", "Vue ESLint 결과 분석", "Vue 린트 검사 결과" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Vue.js ESLint 검사 (eslint-plugin-vue)

`eslint --ext .vue,.ts src/` 실행 결과를 분석하여 vue/* 규칙 위반을 분류하고 자동 수정 가능 여부를 포함한 보고서를 생성한다.
공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "Vue ESLint 검사해줘", "Vue lint 결과 정리", "eslint-plugin-vue 오류 정리"
- "Vue ESLint 결과 분석", "Vue 린트 검사 결과"
- "ESLint 오류 분류해줘 (Vue 프로젝트)"

---

## 입력

### 필수
- `eslint --ext .vue,.ts src/` 실행 결과 (붙여넣기 또는 파일 경로)

### 선택
- `.eslintrc.cjs` 경로 (규칙 설정 확인)
- 프로젝트 소스 경로 (오류 맥락 파악 시 읽기)

---

## 분석 절차

### 1단계: 오류·경고 목록 파싱
ESLint 출력에서 파일·라인·규칙명·심각도(error/warning)·메시지 파싱.

### 2단계: 규칙 유형 분류

| 규칙 접두사 | 분류 | 예시 규칙 |
|:---|:---|:---|
| `vue/` | Vue SFC 규칙 | `vue/require-v-for-key`, `vue/no-unused-vars` |
| `@typescript-eslint/` | TypeScript 규칙 | `@typescript-eslint/no-explicit-any` |
| `no-` / 기타 | ESLint 기본 규칙 | `no-console`, `no-unused-vars` |

주요 vue/* 규칙 설명:
| 규칙 | 설명 | 자동 수정 |
|:---|:---|:---:|
| `vue/require-v-for-key` | `v-for`에 `:key` 없음 | X |
| `vue/no-unused-vars` | template에서 미사용 변수 | X |
| `vue/component-definition-name-casing` | 컴포넌트명 PascalCase 아님 | O |
| `vue/multi-word-component-names` | 단어 하나인 컴포넌트명 | X |
| `vue/no-use-v-if-with-v-for` | `v-if` + `v-for` 동시 사용 | X |
| `vue/script-setup-uses-vars` | `<script setup>` 미사용 변수 | X |
| `vue/html-self-closing` | 자기 닫힘 태그 규칙 | O |
| `vue/max-attributes-per-line` | 줄당 속성 수 초과 | O |

### 3단계: 자동 수정 가능 항목 분리
- `eslint --fix`로 자동 수정 가능한 항목 표시
- 수동 수정 필요 항목 별도 목록

### 4단계: 심각도 분류
- **error**: 즉시 수정 필요 (빌드·배포 차단 가능)
- **warning**: 권장 수정 (코드 품질)

---

## 산출물

`ESLint검사결과_{사업명}_{YYYYMMDD}.md`

### 보고서 구성

```markdown
# Vue.js ESLint 검사 결과

| 항목 | 내용 |
|:---|:---|
| 검사 일자 | YYYY-MM-DD |
| 검사 명령 | `eslint --ext .vue,.ts src/` |
| 총 오류 수 | N건 (error: N / warning: N) |
| 자동 수정 가능 | N건 (`eslint --fix`로 처리) |

## 오류 요약

| 파일 | 라인 | 규칙 | 심각도 | 메시지 | 자동 수정 |
|:---|:---:|:---|:---:|:---|:---:|
| UserTable.vue | 23 | vue/require-v-for-key | error | v-for에 :key 없음 | X |

## 자동 수정 가능 항목

`eslint --ext .vue,.ts src/ --fix` 실행으로 처리:
- N건 자동 수정 예정

## 수동 수정 필요 항목

| 우선순위 | 파일 | 규칙 | 수정 방법 |
|:---:|:---|:---|:---|
| 1 | UserTable.vue | vue/require-v-for-key | :key에 user.id 사용 |
```

## 문서 버전 이력

| 버전 | 일자 | 작성자 | 변경 내용 |
|:---|:---|:---|:---|
| v0.1 | YYYY-MM-DD | 초안 작성 | 최초 생성 |
```
