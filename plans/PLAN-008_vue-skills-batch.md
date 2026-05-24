# PLAN-008: AI-DLC 개발단계(프론트엔드-Vue.js) 스킬 14종 일괄 생성

| 항목 | 내용 |
|:---|:---|
| 작성일 | 2026-05-24 |
| 상태 | 완료 |
| 전제 플랜 | PLAN-006 (ai-dlc-fe-* 스킬 18종), PLAN-007 (ai-dlc-nxt-* 스킬 13종) |
| 스킬 접두사 | `ai-dlc-vue-*` |
| 스킬 경로 | `C:\Users\kdkim2000\.claude\skills\ai-dlc-vue-*\` |

---

## Context

PLAN-006(React/Vite, 18종)·PLAN-007(Next.js, 13종) 완료로 React 계열 프론트엔드 스킬이 갖춰졌다. 프론트엔드 프레임워크 선택지로 **Vue.js 3**를 선택하는 경우에는 별도 스킬 세트가 필요하다.

Vue.js 3와 React의 핵심 차이:
- **SFC(Single File Component)**: `<template>` + `<script setup lang="ts">` + `<style>` 구조 (JSX 없음)
- **Composition API**: `ref()`, `reactive()`, `computed()`, `watch()` (useState/useEffect 대체)
- **Composable**: Custom Hook에 대응하는 Vue 패턴 (`useXxx.ts`)
- **Pinia**: Vue 공식 상태관리 라이브러리 (Zustand 대체)
- **Vue Router v4**: React Router 대체 (`createRouter`, `useRouter`, `useRoute`)
- **VeeValidate v4**: React Hook Form 대체 (Zod 스키마 호환)
- **@tanstack/vue-query**: 동일한 queryKey 패턴 적용 가능 (서버 상태)
- **shadcn-vue / radix-vue**: shadcn/ui의 Vue 포트
- **vue-tsc**: TypeScript 검사 도구 (`tsc` 대신)
- **eslint-plugin-vue**: Vue 전용 ESLint 규칙

`ai-dlc-fe-*` 스킬 중 6종(node-setup, tailwind-guide, axios-guide, zod-guide, e2e-test-validate, e2e-test-revise)은 프레임워크와 무관하므로 재사용한다.

---

## 사용자 요구사항

> `@plans\PLAN-006_fe-react-skills-batch.md 를 참고하여 프론트엔드를 Vue.js 로 선택했을 때 skills를 생성하기 위한 계획을 수립하라. Vue.js 를 선택했을 때도 전체적인 흐름이 이어질 수 있도록 고려하라.`

---

## 설계 결정 사항

| 항목 | 결정 | 근거 |
|:---|:---|:---|
| 총 스킬 수 | 14종 | Vue 고유 기능 커버 + fe-* 6종 재사용 |
| 스킬 접두사 | `ai-dlc-vue-*` | vue = Vue.js; fe-*(React), nxt-*(Next.js)와 명확 구분 |
| Vue.js 버전 | 3 (`<script setup>` + Composition API) | 최신 안정 버전; Vue 2 Options API 미지원 |
| 빌드 도구 | Vite (`npm create vue@latest`) | Vue 공식 권장; PLAN-006 React/Vite와 동일 |
| 상태관리 | Pinia | Vue 공식 권장; Zustand(React) 대응 |
| 라우터 | Vue Router v4 | Vue 공식; React Router 대응 |
| 폼 검증 | VeeValidate v4 + Zod | Zod 스키마 재사용; React Hook Form 대응 |
| 서버 상태 | @tanstack/vue-query | queryKey 패턴 동일; React Query Vue 포트 |
| UI 컴포넌트 | shadcn-vue / radix-vue | shadcn/ui의 Vue 포트; 동일 Tailwind 기반 |
| TypeScript 검사 | vue-tsc (`vue-tsc --noEmit`) | SFC 타입 검사용; tsc 대신 사용 |
| ESLint | eslint-plugin-vue | Vue SFC 전용 규칙 세트 |
| allowed-tools 분류 | 2-tier: create/revise=Read Grep Glob Write Edit, validate/guide=Read Grep Glob | PLAN-005~007 동일 패턴 |
| template.md 보유 스킬 | 4종 (create 그룹 전체) | validate/revise/guide는 콘텐츠가 SKILL.md에 내장 |
| ai-dlc-fe-* 재사용 스킬 | 6종 (새로 생성 안 함) | 프레임워크 무관 내용이므로 중복 생성 불필요 |

---

## ai-dlc-fe-* 재사용 스킬 (새로 생성하지 않음)

| 스킬명 | 재사용 이유 |
|:---|:---|
| `ai-dlc-fe-node-setup` | Express BFF 설정, 프레임워크 무관 |
| `ai-dlc-fe-tailwind-guide` | Tailwind CSS, 프레임워크 무관 |
| `ai-dlc-fe-axios-guide` | Axios HTTP 클라이언트, 프레임워크 무관 |
| `ai-dlc-fe-zod-guide` | Zod 스키마 검증, 프레임워크 무관 |
| `ai-dlc-fe-e2e-test-validate` | EV-001~007 검증 기준 동일 |
| `ai-dlc-fe-e2e-test-revise` | 테스트 수정 패턴 동일 |

---

## 스킬 목록 (14종)

| # | 스킬명 | 역할 | 유형 | allowed-tools |
|:---:|:---|:---|:---:|:---|
| 1 | `ai-dlc-vue-project-setup` | Vue 3 + Vite 프로젝트 초기 설정 생성 | create | Read Grep Glob Write Edit |
| 2 | `ai-dlc-vue-impl-plan` | Vue.js 구현 전략 계획 문서 생성 | create | Read Grep Glob Write Edit |
| 3 | `ai-dlc-vue-component-gen` | SFC 컴포넌트·Composable 코드 생성 | create | Read Grep Glob Write Edit |
| 4 | `ai-dlc-vue-e2e-test-gen` | Playwright e2e 테스트 코드 생성 (Vue용) | create | Read Grep Glob Write Edit |
| 5 | `ai-dlc-vue-code-review` | Vue.js 코드 품질 검토 (SFC·Composition API·Pinia 등) | validate | Read Grep Glob |
| 6 | `ai-dlc-vue-ts-check` | vue-tsc TypeScript 검사 결과 정리 | validate | Read Grep Glob |
| 7 | `ai-dlc-vue-lint-check` | eslint-plugin-vue 검사 결과 정리 | validate | Read Grep Glob |
| 8 | `ai-dlc-vue-code-revise` | Vue.js 코드 리뷰 결과 반영 | revise | Read Grep Glob Write Edit |
| 9 | `ai-dlc-vue-pinia-guide` | Pinia 상태관리 가이드 | guide | Read Grep Glob |
| 10 | `ai-dlc-vue-router-guide` | Vue Router v4 라우팅 가이드 | guide | Read Grep Glob |
| 11 | `ai-dlc-vue-query-guide` | @tanstack/vue-query 서버 상태 가이드 | guide | Read Grep Glob |
| 12 | `ai-dlc-vue-form-guide` | VeeValidate v4 + Zod 폼 검증 가이드 | guide | Read Grep Glob |
| 13 | `ai-dlc-vue-ui-guide` | shadcn-vue / radix-vue UI 컴포넌트 가이드 | guide | Read Grep Glob |
| 14 | `ai-dlc-vue-perf-guide` | Vue.js 성능 최적화 가이드 | guide | Read Grep Glob |

---

## 생성된 파일 구조

```
C:\Users\kdkim2000\.claude\skills\
├── ai-dlc-vue-project-setup\   SKILL.md + template.md  ✅
├── ai-dlc-vue-impl-plan\       SKILL.md + template.md  ✅
├── ai-dlc-vue-component-gen\   SKILL.md + template.md  ✅
├── ai-dlc-vue-e2e-test-gen\    SKILL.md + template.md  ✅
├── ai-dlc-vue-code-review\     SKILL.md                ✅
├── ai-dlc-vue-ts-check\        SKILL.md                ✅
├── ai-dlc-vue-lint-check\      SKILL.md                ✅
├── ai-dlc-vue-code-revise\     SKILL.md                ✅
├── ai-dlc-vue-pinia-guide\     SKILL.md                ✅
├── ai-dlc-vue-router-guide\    SKILL.md                ✅
├── ai-dlc-vue-query-guide\     SKILL.md                ✅
├── ai-dlc-vue-form-guide\      SKILL.md                ✅
├── ai-dlc-vue-ui-guide\        SKILL.md                ✅
└── ai-dlc-vue-perf-guide\      SKILL.md                ✅
```

**총 파일: SKILL.md 14개 + template.md 4개 = 18개**

---

## 스킬별 핵심 설계

### ai-dlc-vue-project-setup
- **트리거**: "Vue.js 프로젝트 만들어줘", "Vue 앱 초기화", "Vite Vue 설정"
- **기술 스택**: Vue 3 + Vite + TypeScript + Pinia + Vue Router v4 + @tanstack/vue-query + VeeValidate v4 + Zod + shadcn-vue + Tailwind CSS + Axios + vue-tsc + eslint-plugin-vue
- **생성 파일**: `package.json`(의존성 전체), `vite.config.ts`(proxy + alias), `tsconfig.json`(strict + verbatimModuleSyntax), `.eslintrc.cjs`(eslint-plugin-vue + typescript), `.prettierrc`, `tailwind.config.ts`, `postcss.config.js`, `src/assets/index.css`(Tailwind + CSS vars), `src/main.ts`(createApp + pinia + router + VueQueryPlugin), `src/App.vue`, `src/router/index.ts`(네비게이션 가드 + 지연 로딩), `src/stores/auth.ts`(Pinia Composition API + persist), `src/lib/axios.ts`(인터셉터 + 401 처리), `src/utils/cn.ts`, `src/types/auth.ts`, `.env.example`
- **디렉터리**: `src/views/`, `src/components/`, `src/composables/`, `src/stores/`, `src/router/`, `src/types/`, `src/api/`

### ai-dlc-vue-impl-plan
- **트리거**: "Vue.js 구현 계획 세워줘", "Vue 화면 구현 순서", "Vue 컴포넌트 설계"
- **절차**: 화면 목록(SCR-NNN) → 라우트 구조 설계 → 컴포넌트 분류(View/Container/Presentational) → Pinia 스토어 목록(UI 상태만) → Composable 목록 → API 모듈 목록 → 구현 우선순위
- **산출물**: `Vue구현계획_{사업명}_{YYYYMMDD}.md` (화면 목록 표, 라우트 구조 표, 컴포넌트 분류 표, Pinia 스토어 표, Composable 표, API 모듈 표)

### ai-dlc-vue-component-gen
- **트리거**: "Vue 컴포넌트 만들어줘", "SFC 생성", "Composable 만들어줘", "Vue 화면 구현"
- **원칙**: `<script setup lang="ts">` 필수 / 컴포넌트 max 150줄 / 로직은 Composable로 추출 / props는 `defineProps<T>()` / emits는 `defineEmits<T>()` / `v-for`는 반드시 `:key` (index 사용 금지)
- **생성 파일**: `src/views/UsersView.vue`(목록 View + useQuery), `src/components/UserTable.vue`(Presentational), `src/components/UserForm.vue`(VeeValidate + Zod), `src/composables/useUsers.ts`(useQuery/useMutation 래핑), `src/stores/userStore.ts`(Pinia, UI 상태만), `src/api/user.api.ts`, `src/types/user.types.ts`

### ai-dlc-vue-e2e-test-gen
- **트리거**: "Vue e2e 테스트 만들어줘", "Vue Playwright 테스트", "Vue 통합 테스트 생성"
- **특화**: `webServer: { command: 'npm run dev', port: 3000 }`, 인증 픽스처(`authenticatedPage`), VeeValidate 비동기 검증 대기(`waitForSelector`), Vue Router URL 구조, `data-testid` 선택자
- **생성 파일**: `playwright.config.ts`, `tests/fixtures/auth.ts`, `tests/pages/BasePage.ts`, `tests/pages/UsersPage.ts` + `CreateUserPage`, `tests/specs/users.spec.ts`, `tests/specs/auth.spec.ts`

### ai-dlc-vue-code-review
- **트리거**: "Vue 코드 검토해줘", "SFC 코드 리뷰", "Vue 컴포넌트 리뷰"
- **이슈 코드**: VV-001~010 (Vue.js 고유) + TC/PF/SC/A11Y (공통)
- **높음 심각도**: VV-003(Pinia 없이 상태 공유), VV-008(v-for :key 미설정/index 사용), VV-009(컴포넌트에서 API 직접 호출), VV-010(VeeValidate 미사용 폼)
- **산출물**: `코드품질검토_{사업명}_{YYYYMMDD}.md`

### ai-dlc-vue-ts-check
- **트리거**: "Vue TypeScript 검사해줘", "vue-tsc 실행", "SFC 타입 오류 정리"
- **명령**: `vue-tsc --noEmit`
- **오류 분류**: SFC props 타입 불일치 / Composable 반환 타입 누락 / ref/reactive 제네릭 미지정 / Pinia storeToRefs 타입 손실 / Vue Router useRoute 타입 캐스팅 / 기타 TS 오류
- **산출물**: `TypeScript검사결과_{사업명}_{YYYYMMDD}.md`

### ai-dlc-vue-lint-check
- **트리거**: "Vue ESLint 검사해줘", "Vue lint 결과 정리", "eslint-plugin-vue 오류 정리"
- **명령**: `eslint --ext .vue,.ts src/`
- **vue/* 규칙**: vue/no-unused-vars, vue/require-v-for-key, vue/component-definition-name-casing, vue/no-mutating-props, vue/multi-word-component-names 등
- **산출물**: `ESLint검사결과_{사업명}_{YYYYMMDD}.md`

### ai-dlc-vue-code-revise
- **트리거**: "Vue 코드 수정해줘", "Vue 리뷰 결과 반영", "VV 이슈 코드 수정"
- **수정 우선순위**: VV-003/VV-008/VV-009/VV-010(아키텍처·런타임 오류) → VV-001/VV-002(코드 스타일) → TC(타입) → VV-004/VV-005 → 나머지
- **수정 패턴**: `v-for index→data.id` / `axios 직접→Composable+useQuery` / `직접 submit→VeeValidate+Zod` / `Options API→<script setup>` / `untyped props→defineProps<T>()`

### ai-dlc-vue-pinia-guide
- **트리거**: "Pinia 사용법", "Vue 상태관리 방법", "Pinia 스토어 만들기"
- **내용**: `defineStore` Composition API 패턴 / `storeToRefs`로 반응성 유지 (필수) / 스토어 간 참조 / `pinia-plugin-persistedstate`로 localStorage 영속성 / 설계 원칙(서버 상태→Vue Query, UI 상태→Pinia, 인증→Pinia+persist)

### ai-dlc-vue-router-guide
- **트리거**: "Vue Router 사용법", "Vue 라우팅 설정", "네비게이션 가드", "동적 라우트"
- **내용**: `createRouter` + `createWebHistory` / `useRouter()`·`useRoute()` / `beforeEach` 가드 / `RouteMeta` 타입 확장(`router.d.ts`) / `onBeforeRouteLeave` / 동적 라우트(Number 변환 주의) / 지연 로딩 / `RouterLink` active-class / 중첩 라우트

### ai-dlc-vue-query-guide
- **트리거**: "Vue Query 사용법", "useQuery 설정", "vue-query 데이터 패칭"
- **내용**: `VueQueryPlugin` 등록 + `QueryClient` 옵션 / `useQuery`(reactive queryKey via `computed()`) / `userKeys` 계층 패턴 / `useMutation`(invalidateQueries, setQueryData, removeQueries) / `prefetchQuery` / Vue Query DevTools(`VueQueryDevtools` 컴포넌트) / Vue Query vs Pinia 구분 표

### ai-dlc-vue-form-guide
- **트리거**: "VeeValidate 사용법", "Vue 폼 검증", "Zod Vue 폼"
- **내용**: `useForm({ validationSchema: toTypedSchema(zodSchema) })` / `useField` vs `Field` 컴포넌트 / `handleSubmit(onValid)` 패턴 / `setValues`(수정 폼) / 비동기 `refine` 검증 / shadcn-vue Input/Select 연동 / `resetForm`·`setErrors`·`setFieldError` / 자주 쓰는 Zod 패턴(optional, nullable, coerce.number, regex, superRefine)

### ai-dlc-vue-ui-guide
- **트리거**: "shadcn-vue 설치", "radix-vue 컴포넌트", "Vue UI 라이브러리 설정"
- **내용**: shadcn-vue CLI 초기화 + `add` 명령 / Button(variant/size/loading spinner) / Dialog(ConfirmDialog 패턴) / Select / Table(빈 상태 colspan) / 다크모드(`useColorMode`) / Card / Badge(동적 variant) / Tailwind CSS 변수 테마

### ai-dlc-vue-perf-guide
- **트리거**: "Vue 성능 최적화", "Vue 렌더링 최적화", "Vite 번들 최적화"
- **내용**: `defineAsyncComponent`(loadingComponent, errorComponent, delay, timeout) / `v-memo`(의존성 배열) / `shallowRef` vs `ref` 기준(대형 배열·read-only → shallowRef + Object.freeze) / `KeepAlive`(:include, :max, RouterView 슬롯, route meta keepAlive, onActivated/onDeactivated) / `computed` 최적화 / Vite: `rollup-plugin-visualizer` + `manualChunks`(vendor-vue/query/form/ui) / 이미지 lazy loading

---

## 검증 방법

| 트리거 문장 | 기대 스킬 |
|:---|:---|
| "Vue.js 프로젝트 만들어줘" | `ai-dlc-vue-project-setup` |
| "Vue 구현 계획 세워줘" | `ai-dlc-vue-impl-plan` |
| "사용자 목록 Vue 컴포넌트 만들어줘" | `ai-dlc-vue-component-gen` |
| "Vue Playwright 테스트 생성" | `ai-dlc-vue-e2e-test-gen` |
| "Vue 코드 리뷰해줘" | `ai-dlc-vue-code-review` |
| "vue-tsc TypeScript 검사해줘" | `ai-dlc-vue-ts-check` |
| "Vue ESLint 검사 결과 정리해줘" | `ai-dlc-vue-lint-check` |
| "Vue 코드 리뷰 결과 반영해줘" | `ai-dlc-vue-code-revise` |
| "Pinia 스토어 사용법 알려줘" | `ai-dlc-vue-pinia-guide` |
| "Vue Router 네비게이션 가드 설정법" | `ai-dlc-vue-router-guide` |
| "Vue Query useQuery 사용법" | `ai-dlc-vue-query-guide` |
| "VeeValidate Zod 폼 검증 방법" | `ai-dlc-vue-form-guide` |
| "shadcn-vue 컴포넌트 설치 방법" | `ai-dlc-vue-ui-guide` |
| "Vue 성능 최적화 방법" | `ai-dlc-vue-perf-guide` |

---

## 공유 참조 파일 업데이트

| 파일 | 업데이트 내용 |
|:---|:---|
| `quality-checklist.md` | VV-001~010 이슈 코드 표 추가 (개발단계-프론트엔드-Vue.js 섹션) |
| `artifacts-flow.md` | Vue.js 산출물 흐름 섹션 추가 (ASCII 다이어그램 + 11단계 스킬 사용 순서) |

---

## 비범위

- Nuxt.js (SSR/SSG Vue 프레임워크 — 별도 PLAN 고려)
- Vue 2 / Options API 중심 코드 (지원 종료)
- React Native / Expo (모바일)
- Svelte/SvelteKit, Angular
- Vuex (Pinia로 대체됨, 신규 프로젝트 미지원)
- GraphQL / Apollo Vue 연동
- Storybook Vue 컴포넌트 문서화
- CI/CD 파이프라인 (GitHub Actions, Jenkins)
- 단위 테스트 (Vitest + Vue Test Utils) — 별도 PLAN 고려
