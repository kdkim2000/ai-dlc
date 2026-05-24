---
name: ai-dlc-vue-code-review
description: AI-DLC 개발단계(프론트엔드-Vue.js) 스킬. Vue.js SFC 코드 품질을 검토하고 이슈 코드(VV-001~010)로 분류된 보고서를 생성한다. "Vue 코드 검토해줘", "SFC 코드 리뷰", "Vue 컴포넌트 리뷰", "뷰 코드 품질 검토", "Vue 코드 품질 분석", "Vue 코드 리뷰해줘" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Vue.js 코드 품질 검토

Vue.js SFC 코드를 분석하여 VV-001~010(Vue 고유) + TC/PF/SC/A11Y(공통) 이슈를 발견하고 코드품질검토 보고서를 생성한다.
공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "Vue 코드 검토해줘", "SFC 코드 리뷰", "Vue 컴포넌트 리뷰"
- "뷰 코드 품질 검토", "Vue 코드 품질 분석", "Vue 코드 리뷰해줘"
- "VV 이슈 확인해줘", "Vue Composition API 리뷰"

---

## 입력

### 필수
- 검토 대상 소스 파일 경로 또는 디렉터리 (`.vue`, `.ts` 파일)

### 선택
- 검토 범위 (특정 컴포넌트, 도메인, 전체 프로젝트)
- 집중 검토 항목 (보안, 성능, 타입 등)

---

## 검사 항목

### Vue.js 고유 이슈 코드 (VV-001~010)

| 코드 | 검사 항목 | 심각도 | 검사 방법 |
|:---|:---|:---:|:---|
| VV-001 | Options API 사용 — `<script setup>`으로 전환 권장 | 중간 | `export default {` 패턴 탐지 |
| VV-002 | `<script setup>` 미사용 — 장황한 `setup()` 반환 패턴 | 중간 | `setup() {` + `return {` 패턴 탐지 |
| VV-003 | Pinia 없이 컴포넌트 간 상태 공유 (props drilling 2단계 초과) | 높음 | prop 체인 깊이 분석 |
| VV-004 | 로직이 컴포넌트 내 직접 작성 — Composable 미추출 | 중간 | 컴포넌트 내 API 호출·복잡 로직 탐지 |
| VV-005 | `defineProps`/`defineEmits` 타입 미정의 | 중간 | `defineProps()` 비제네릭 호출 탐지 |
| VV-006 | `watch` 대신 `watchEffect` 남용 (의존성 불명확) | 낮음 | `watchEffect` 사용 빈도 분석 |
| VV-007 | `$router`/`$route` 직접 접근 — `useRouter()`/`useRoute()` 미사용 | 낮음 | `this.$router`, `this.$route` 패턴 탐지 |
| VV-008 | `v-for`에 `:key` 미설정 또는 index를 key로 사용 | 높음 | `v-for` 없는 `:key`, `:key="index"` 탐지 |
| VV-009 | 컴포넌트에서 API 직접 호출 — Vue Query/API 모듈 미사용 | 높음 | SFC 내 `axios.`/`fetch(` 직접 호출 탐지 |
| VV-010 | 폼 검증 없이 직접 submit — VeeValidate 미사용 | 높음 | `@submit`/`submit` 핸들러에 검증 로직 없음 탐지 |

### 공통 이슈 코드

| 코드 | 검사 항목 | 심각도 |
|:---|:---|:---:|
| TC-001 | TypeScript `any` 타입 사용 | 높음 |
| TC-002 | 타입 단언(`as`) 과다 사용 | 중간 |
| TC-003 | props/emits 타입 미정의 (VV-005와 중복 시 VV-005 우선) | 중간 |
| PF-001 | 불필요한 `ref` 중첩 (ref 안에 ref) | 중간 |
| PF-002 | `v-if` + `v-for` 동시 사용 (성능 저하) | 중간 |
| PF-003 | `computed` 미사용 — template 내 복잡 표현식 | 낮음 |
| SC-001 | 하드코딩된 인증 토큰·비밀번호·API Key | 높음 |
| SC-002 | `innerHTML`/`v-html`에 미검증 사용자 입력 (XSS) | 높음 |
| A11Y-001 | 인터랙티브 요소에 `aria-label` 누락 | 낮음 |
| A11Y-002 | 이미지에 `alt` 속성 누락 | 낮음 |

---

## 검토 절차

### 1단계: 파일 목록 수집
대상 경로의 `.vue`, `.ts` 파일을 Glob으로 수집.

### 2단계: 파일별 정적 분석
각 파일에서:
- Script 블록 분석 (`<script setup lang="ts">` 여부)
- Template 블록 분석 (v-for :key, v-html, aria 등)
- 컴포넌트 구조 분석 (로직 분리 여부)

### 3단계: 이슈 목록 작성
발견된 이슈를 코드·파일·라인 기준으로 정리.

### 4단계: 보고서 생성
- 파일명: `코드품질검토_{사업명}_{YYYYMMDD}.md`
- 이슈를 심각도(높음 → 중간 → 낮음) 순 정렬
- 각 이슈에 수정 방향 제시

---

## 산출물

`코드품질검토_{사업명}_{YYYYMMDD}.md`

### 보고서 구성

```markdown
# Vue.js 코드 품질 검토 보고서

| 항목 | 내용 |
|:---|:---|
| 검토 일자 | YYYY-MM-DD |
| 검토 범위 | src/views/, src/components/ |
| 총 이슈 수 | N건 (높음: N / 중간: N / 낮음: N) |

## 이슈 요약

| 코드 | 파일 | 라인 | 심각도 | 설명 |
|:---|:---|:---:|:---:|:---|
| VV-008 | UserTable.vue | 23 | 높음 | v-for :key에 index 사용 |
| VV-009 | UsersView.vue | 15 | 높음 | 컴포넌트 내 axios 직접 호출 |

## 이슈 상세

### [VV-009] 컴포넌트 내 API 직접 호출
**파일**: `src/views/UsersView.vue:15`
**현재 코드**: ...
**수정 방향**: useQuery + Composable로 분리
```

## 문서 버전 이력

| 버전 | 일자 | 작성자 | 변경 내용 |
|:---|:---|:---|:---|
| v0.1 | YYYY-MM-DD | 초안 작성 | 최초 생성 |
```
