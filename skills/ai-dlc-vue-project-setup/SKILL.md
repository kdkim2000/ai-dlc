---
name: ai-dlc-vue-project-setup
description: AI-DLC 개발단계(프론트엔드-Vue.js) 스킬. Vue 3 + Vite + TypeScript 프로젝트 초기 설정 파일 일체를 생성한다. "Vue.js 프로젝트 만들어줘", "Vue 앱 초기화", "Vite Vue 설정", "Vue 프로젝트 초기화", "Vue3 프로젝트 셋업", "뷰 프로젝트 생성", "프론트엔드 Vue 초기 설정" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Vue 3 + Vite 프로젝트 초기 설정

설계단계 산출물(화면설계서·API설계서)을 기반으로 Vue 3 + Vite + TypeScript 프로젝트 초기 설정 파일 일체를 생성한다.
공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "Vue.js 프로젝트 만들어줘", "Vue 앱 초기화", "Vite Vue 설정"
- "Vue 프로젝트 초기화", "Vue3 프로젝트 셋업", "뷰 프로젝트 생성"
- "Vue 프론트엔드 초기 설정", "create vue 설정"

---

## 입력

### 필수
- 프로젝트명 (디렉터리명·package.json name)
- API 서버 URL (vite.config.ts proxy 설정)

### 선택
- 화면설계서(SCR-NNN) — 라우팅 경로 추출 시 활용
- 인증 방식 (JWT 기본값)

---

## 분석 절차

### 1단계: 기술 스택 파악
사용자가 언급한 스택을 정리하고, 미언급 항목은 기본값 적용.

기본 스택:
- Vue 3.x + `<script setup lang="ts">` + Composition API
- 빌드: Vite 5.x
- 상태 관리: Pinia (Vue 공식 권장)
- 라우터: Vue Router v4
- 서버 상태: @tanstack/vue-query
- HTTP: Axios
- 폼 검증: VeeValidate v4 + Zod
- UI: shadcn-vue / radix-vue + Tailwind CSS
- TypeScript: vue-tsc strict

### 2단계: 의존성 구성 (package.json)

**dependencies**:
- 코어: `vue`, `vue-router`, `pinia`
- 서버 상태: `@tanstack/vue-query`, `@tanstack/vue-query-devtools`
- HTTP: `axios`
- 폼 검증: `vee-validate`, `@vee-validate/zod`, `zod`
- UI: `radix-vue`, `class-variance-authority`, `clsx`, `tailwind-merge`, `lucide-vue-next`
- Pinia 영속성: `pinia-plugin-persistedstate`

**devDependencies**:
- 빌드: `vite`, `@vitejs/plugin-vue`
- TypeScript: `typescript`, `vue-tsc`
- CSS: `tailwindcss`, `postcss`, `autoprefixer`
- 린트: `eslint`, `eslint-plugin-vue`, `@typescript-eslint/eslint-plugin`, `@typescript-eslint/parser`, `prettier`, `eslint-config-prettier`
- e2e: `@playwright/test`

### 3단계: TypeScript 설정 (tsconfig.json)
- `strict: true` 필수
- `baseUrl: "."`, `paths: { "@/*": ["src/*"] }` — 절대경로 import
- `target: "ES2020"`, `lib: ["ES2020", "DOM", "DOM.Iterable"]`
- `verbatimModuleSyntax: true` — Vue SFC 타입 안전성

### 4단계: ESLint + Prettier 설정
- `.eslintrc.cjs`: eslint-plugin-vue (vue3-recommended) + typescript-eslint
  - `vue/no-unused-vars: error`
  - `vue/require-v-for-key: error`
  - `vue/component-definition-name-casing: ["error", "PascalCase"]`
  - `vue/script-setup-uses-vars: error`
  - `@typescript-eslint/no-explicit-any: error`
- `.prettierrc`: `singleQuote: true`, `trailingComma: "all"`, `semi: false`, `printWidth: 100`

### 5단계: Tailwind CSS + shadcn-vue 설정
- `tailwind.config.ts`: content 경로(`.vue` 파일 포함), `darkMode: "class"`, CSS 변수 기반 컬러 팔레트
- `postcss.config.js`: tailwindcss + autoprefixer
- `src/assets/index.css`: Tailwind base/components/utilities + CSS 변수 정의

### 6단계: 디렉터리 구조 생성
```
src/
├── views/          # 페이지 컴포넌트 (라우트 단위, *View.vue)
├── components/     # 공통·재사용 컴포넌트
│   └── ui/         # shadcn-vue 생성 컴포넌트
├── composables/    # Composable (useXxx.ts)
├── stores/         # Pinia 스토어 (도메인별)
├── router/         # Vue Router 설정
│   └── index.ts    # createRouter + 네비게이션 가드
├── api/            # API 클라이언트 함수 (도메인별)
├── types/          # TypeScript 타입 정의
├── utils/          # 순수 유틸리티 함수
├── lib/            # 라이브러리 초기화 (axios 등)
├── App.vue         # 루트 컴포넌트 (RouterView)
├── main.ts         # 앱 진입점 (Pinia + Router + VueQuery 등록)
└── assets/
    └── index.css   # Tailwind 진입점
```

### 7단계: 핵심 설정 파일 생성
- `src/lib/axios.ts`: Axios 인스턴스, 요청·응답 인터셉터(토큰 주입, 401 처리)
- `src/lib/queryClient.ts`: QueryClient 옵션 설정 (staleTime 5분, gcTime 10분)
- `src/utils/cn.ts`: `clsx + tailwind-merge` 조합 `cn()` 함수
- `src/router/index.ts`: Vue Router v4, 네비게이션 가드(인증 체크), 지연 로딩
- `src/stores/auth.ts`: Pinia auth 스토어 (토큰, 사용자 정보, login/logout actions)
- `src/App.vue`: RouterView + RouterLink 루트 레이아웃
- `src/main.ts`: 앱 진입점 (createApp + use 체인)

---

## 생성 원칙

- **경로 alias**: `@/` → `src/` 통일 (tsconfig + vite.config 동기화)
- **환경 변수**: `.env.example` 생성, `.env`는 `.gitignore` 처리. `import.meta.env.VITE_*` 사용
- **인라인 스타일 금지**: Tailwind 유틸리티 클래스 사용
- **cn() 함수**: 모든 조건부 클래스 조합에 `cn()` 사용
- **SFC 순서**: `<script setup>` → `<template>` → `<style scoped>` 순서 준수

---

## 산출물

| 파일 | 설명 |
|:---|:---|
| `package.json` | 의존성·스크립트 전체 |
| `vite.config.ts` | 빌드·proxy·alias 설정 |
| `tsconfig.json` + `tsconfig.node.json` | TypeScript strict 설정 |
| `.eslintrc.cjs` | ESLint(eslint-plugin-vue + typescript) 규칙 |
| `.prettierrc` | Prettier 포맷 설정 |
| `tailwind.config.ts` | Tailwind 커스텀 설정 |
| `postcss.config.js` | PostCSS 플러그인 |
| `src/assets/index.css` | Tailwind + CSS 변수 |
| `src/main.ts` | 앱 진입점 |
| `src/App.vue` | 루트 컴포넌트 |
| `src/router/index.ts` | Vue Router 설정 + 네비게이션 가드 |
| `src/stores/auth.ts` | Pinia auth 스토어 |
| `src/lib/axios.ts` | Axios 인스턴스 + 인터셉터 |
| `src/lib/queryClient.ts` | Vue Query 클라이언트 옵션 |
| `src/utils/cn.ts` | cn() 유틸리티 |
| `.env.example` | 환경 변수 예시 |

template.md에서 각 파일의 기본 코드 골격을 참조한다.
