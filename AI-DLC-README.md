# AI-DLC 개발 가이드

> **AI-DLC(AI-Driven Development Lifecycle)** — Claude Code 스킬을 활용하여 요구사항 정의부터 코드 생성·검증까지 전체 개발 생명주기를 AI와 함께 진행하는 방법론

---

## 목차

1. [전체 프로세스 개요](#1-전체-프로세스-개요)
2. [아이디어 단계 (요구사항이 불명확할 때)](#2-아이디어-단계-요구사항이-불명확할-때)
3. [빠른 시작 (처음 사용자용)](#3-빠른-시작-처음-사용자용)
4. [단계별 스킬 가이드](#4-단계별-스킬-가이드)
   - [요구사항 정의](#31-요구사항-정의)
   - [분석](#32-분석)
   - [설계](#33-설계)
   - [개발 — 아키텍처 분기](#34-개발--아키텍처-분기)
5. [아키텍처별 개발 경로](#5-아키텍처별-개발-경로)
   - [Spring Boot 백엔드](#51-spring-boot-백엔드)
   - [React/Vite 프론트엔드](#52-reactvite-프론트엔드)
   - [Next.js App Router](#53-nextjs-app-router)
   - [Vue.js 3](#54-vuejs-3)
6. [스킬 전체 목록](#6-스킬-전체-목록)
7. [ID 체계 및 산출물 연계](#7-id-체계-및-산출물-연계)
8. [산출물 파일 규칙](#8-산출물-파일-규칙)
9. [스킬 사용법](#9-스킬-사용법)

---

## 1. 전체 프로세스 개요

```
┌─────────────────────────────────────────────────────────────────────┐
│                        AI-DLC 전체 흐름                              │
│                                                                     │
│  [요구사항 정의]  →  [분석]  →  [설계]  →  [개발]  →  [검증]         │
│                                                                     │
│  요구사항 정의서       유즈케이스          코드 생성     e2e 테스트   │
│  서비스 카탈로그       화면설계서          단위 테스트   품질 검토    │
│  비즈니스 규칙         API 설계서          코드 리뷰                 │
│  도메인 용어사전       데이터 설계서                                  │
│                        클래스 설계서                                  │
└─────────────────────────────────────────────────────────────────────┘
```

각 단계는 이전 단계의 산출물을 입력으로 받아 다음 단계의 산출물을 생성한다. Claude Code 채팅창에서 스킬 트리거 문장을 입력하면 해당 스킬이 자동으로 실행된다.

---

## 2. 아이디어 단계 (요구사항이 불명확할 때)

요구사항을 모르지만 **막연한 불편함·아이디어가 있을 때** 사용하는 사전 단계 스킬이다.

```
[아이디어 / 불편함 / 막연한 필요]
          │
          ▼
  ai-dlc-idea-clarify          ← "아이디어 구체화해줘"
  (아이디어정의서_*.md)
          │
     ┌────┴────────────────┐
     ▼                     ▼
ai-dlc-persona-create   ai-dlc-user-story-map
"페르소나 만들어줘"       "사용자 스토리 만들어줘"
     └────────────────────┘
                │
                ▼
      ai-dlc-mvp-scope         ← "MVP 정의해줘"
      (MVP범위정의서_*.md)
                │
                ▼
      ai-dlc-idea-to-req       ← "요구사항으로 변환해줘"
      (요구사항정의서_*.md)
                │
                ▼
      [기존 AI-DLC 파이프라인으로 연결]
```

| 스킬 | 트리거 예시 | 산출물 |
|:---|:---|:---|
| `ai-dlc-idea-clarify` | "앱 만들고 싶어", "이런 게 불편해" | 아이디어정의서_*.md |
| `ai-dlc-persona-create` | "페르소나 만들어줘" | 페르소나_*.md (PS-NNN) |
| `ai-dlc-user-story-map` | "사용자 스토리 만들어줘", "US 만들어줘" | 사용자스토리맵_*.md (US-NNN) |
| `ai-dlc-mvp-scope` | "MVP 정의해줘", "MoSCoW 분류해줘" | MVP범위정의서_*.md |
| `ai-dlc-idea-to-req` | "요구사항으로 변환해줘" | 요구사항정의서_*.md (FR-NNN) |

> **이미 요구사항이 명확하다면** 이 단계를 건너뛰고 바로 `ai-dlc-requirements`를 사용한다.

---

## 3. 빠른 시작 (처음 사용자용)

### 스킬 실행 방법

```
Claude Code 채팅창에 자연어로 입력:

예) "요구사항 정의서 작성해줘"
    "유즈케이스 만들어줘"
    "Spring Boot 프로젝트 설정해줘"
    "Next.js 페이지 만들어줘"
```

스킬은 자동으로 트리거 문장을 인식하고 실행된다. 별도의 명령어나 `/스킬명` 형식이 필요 없다.

### 신규 프로젝트 최소 경로 (5단계)

```
1. "요구사항 정의서 작성해줘" (ai-dlc-requirements)
        ↓
2. "유즈케이스 만들어줘" (ai-dlc-usecase-create)
        ↓
3. "화면 목록 만들어줘 + API 설계해줘" (ai-dlc-screen-list + ai-dlc-api-design)
        ↓
4. [아키텍처 선택 후] 프로젝트 설정 및 코드 생성
        ↓
5. "코드 리뷰해줘 + e2e 테스트 만들어줘"
```

### 아키텍처 선택 기준

| 상황 | 선택 |
|:---|:---|
| Java 백엔드 REST API | Spring Boot (`ai-dlc-sb-*`) |
| React SPA (Vite, 별도 API 서버 연동) | React/Vite (`ai-dlc-fe-*`) |
| React + SSR/SSG + 인증 + 풀스택 | Next.js App Router (`ai-dlc-nxt-*`) |
| 풀스택 단일 프레임워크 | Next.js (`ai-dlc-nxt-*`) |
| Vue.js SPA (Pinia, Vue Router v4 기반) | Vue.js 3 (`ai-dlc-vue-*`) |

---

## 3. 단계별 스킬 가이드

### 3.1 요구사항 정의

**목표**: 프로젝트의 요구사항을 정형화된 문서로 정리

```
입력: 기획서, 회의록, 인터뷰 내용, 아이디어
출력: 요구사항 정의서 (FR/PR/SR/QR/IR/DR/CR-NNN)
```

| 순서 | 스킬 | 트리거 예시 | 산출물 |
|:---:|:---|:---|:---|
| 1 | `ai-dlc-requirements` | "요구사항 정의서 작성해줘" | 요구사항정의서_*.md |
| 2 | `ai-dlc-service-catalog` | "서비스 카탈로그 만들어줘" | 서비스분류_*.md |
| 3 | `ai-dlc-functional-req` | "기능 요구사항 정의해줘" | 기능요구사항_*.md |
| 4 | `ai-dlc-nonfunctional-req` | "비기능 요구사항 정의해줘" | 비기능요구사항_*.md |

---

### 3.2 분석

**목표**: 요구사항에서 비즈니스 규칙과 도메인 용어를 도출·표준화

```
입력: 요구사항 정의서, 서비스 카탈로그
출력: 비즈니스 규칙 테이블, 도메인 표준 용어사전
```

| 순서 | 스킬 | 트리거 예시 | 산출물 |
|:---:|:---|:---|:---|
| 1 | `ai-dlc-biz-rules-create` | "비즈니스 규칙 도출해줘" | 비즈니스규칙_*.md |
| 2 | `ai-dlc-biz-rules-validate` | "비즈니스 규칙 검증해줘" | (검증 보고서) |
| 3 | `ai-dlc-biz-rules-revise` | "비즈니스 규칙 수정해줘" | 비즈니스규칙_*_v0.2.md |
| 4 | `ai-dlc-glossary-create` | "도메인 용어사전 만들어줘" | 용어사전_*.md |
| 5 | `ai-dlc-glossary-validate` | "용어사전 검증해줘" | (검증 보고서) |
| 6 | `ai-dlc-glossary-revise` | "용어사전 수정해줘" | 용어사전_*_v0.2.md |
| 7 | `ai-dlc-glossary-apply` | "요구사항에 표준 용어 반영해줘" | 요구사항정의서_*_v2.md |

**코드 분석 (기존 시스템 분석 시)**

| 스킬 | 트리거 예시 | 산출물 |
|:---|:---|:---|
| `ai-dlc-api-spec-extract` | "API 명세 추출해줘" | OpenAPI YAML |
| `ai-dlc-data-model-analysis` | "DB 스키마 분석해줘" | 데이터모델분석_*.md |
| `ai-dlc-code-traceability` | "코드 추적성 분석해줘" | 추적성분석_*.md |
| `ai-dlc-dependency-analysis` | "의존성 분석해줘" | 의존성분석_*.md |
| `ai-dlc-code-complexity` | "코드 복잡도 분석해줘" | 복잡도분석_*.md |
| `ai-dlc-design-extract` | "레거시 설계 추출해줘" | 설계기초추출서_*.md |

---

### 3.3 설계

**목표**: 유즈케이스 → 화면·API·데이터·클래스 설계서 생성

```
입력: 요구사항(FR-NNN), 서비스 카탈로그(SC-NNN), 도메인 용어사전
출력: UC-NNN, SCR-NNN, API YAML, 데이터설계서, 클래스설계서, 시퀀스다이어그램
```

#### 설계 흐름

```
ai-dlc-usecase-create (UC-NNN 유즈케이스)
        │
   ┌────┴─────────────────────────────────────┐
   ▼                                           ▼
ai-dlc-screen-list (SCR 목록)          ai-dlc-api-design (OpenAPI)
ai-dlc-screen-spec (화면 상세)         ai-dlc-api-validate
ai-dlc-screen-validate                 ai-dlc-api-revise 반복
ai-dlc-screen-revise 반복
        │                                      │
        └──────────────┬───────────────────────┘
                       ▼
              ai-dlc-data-design → ai-dlc-data-validate → ai-dlc-data-revise
                       │
                       ▼
              ai-dlc-class-design → ai-dlc-class-validate → ai-dlc-class-revise
                       │
                       ▼
              ai-dlc-sequence-design (선택)
```

| 스킬 | 트리거 예시 | 산출물 |
|:---|:---|:---|
| `ai-dlc-usecase-create` | "유즈케이스 만들어줘" | 유즈케이스_*.md |
| `ai-dlc-usecase-validate` | "유즈케이스 검증해줘" | (검증 보고서) |
| `ai-dlc-screen-list` | "화면 목록 만들어줘" | 화면목록_*.md |
| `ai-dlc-screen-spec` | "화면설계서 만들어줘" | 화면설계서_*.md |
| `ai-dlc-screen-validate` | "화면설계 검증해줘" | (검증 보고서) |
| `ai-dlc-api-design` | "API 설계해줘" | openapi_*.yaml |
| `ai-dlc-api-validate` | "API 설계 검증해줘" | (검증 보고서) |
| `ai-dlc-data-design` | "데이터 설계해줘" | 데이터설계서_*.md |
| `ai-dlc-class-design` | "클래스 설계해줘" | 클래스설계서_*.md |
| `ai-dlc-sequence-design` | "시퀀스 다이어그램 만들어줘" | 시퀀스다이어그램_*.md |

---

### 3.4 개발 — 아키텍처 분기

설계 단계 완료 후, 프로젝트 아키텍처에 따라 개발 경로가 분기된다.

```
[설계 완료: UC-NNN, SCR-NNN, API YAML, 데이터설계서, 클래스설계서]
                          │
      ┌────────────┼────────────────┬──────────────┐
      ▼            ▼                ▼              ▼
[Spring Boot]  [React/Vite]    [Next.js]      [Vue.js 3]
ai-dlc-sb-*    ai-dlc-fe-*    ai-dlc-nxt-*   ai-dlc-vue-*
```

---

## 4. 아키텍처별 개발 경로

### 4.1 Spring Boot 백엔드

**적합한 경우**: Java 기반 REST API 서버, Anyframe 프레임워크 활용

#### 개발 흐름

```
ai-dlc-sb-project-setup (또는 ai-dlc-sb-anyframe-setup)
        │
   ┌────┴──────────────────────────────────────┐
   ▼                                           ▼
ai-dlc-sb-migration-plan               프로젝트 설정 완료 후
ai-dlc-sb-sql-plan                     ai-dlc-sb-layer-impl
ai-dlc-sb-sql-gen                      (구현 순서 안내)
ai-dlc-sb-migration-exec                      │
(DB 스키마 생성)               ┌──────────────┼────────────────┐
                               ▼              ▼                ▼
                         ai-dlc-sb-vo-gen  ai-dlc-sb-mapper-gen
                         (*VO.java)        (*Mapper.java + XML)
                                                    │
                                                    ▼
                                         ai-dlc-sb-service-gen
                                         (*Service.java + Impl)
                                                    │
                                                    ▼
                                         ai-dlc-sb-controller-gen
                                         (*Controller.java)
                                                    │
                               ┌────────────────────┴───────────┐
                               ▼                                ▼
                    ai-dlc-sb-unit-test-gen          ai-dlc-sb-code-review
                    → ai-dlc-sb-unit-test-validate   → ai-dlc-sb-code-revise
                    → ai-dlc-sb-unit-test-revise
```

#### 단계별 스킬 상세

| 순서 | 스킬 | 트리거 예시 | 산출물 |
|:---:|:---|:---|:---|
| 1 | `ai-dlc-sb-project-setup` | "Spring Boot 프로젝트 설정해줘" | pom.xml, application-*.yml |
| 1a | `ai-dlc-sb-anyframe-setup` | "Anyframe 설정해줘" | Anyframe 설정 파일 (선택) |
| 2 | `ai-dlc-sb-migration-plan` | "DB 마이그레이션 계획 세워줘" | DB마이그레이션계획_*.md |
| 3 | `ai-dlc-sb-sql-plan` | "SQL 생성 계획 세워줘" | SQL생성계획_*.md |
| 4 | `ai-dlc-sb-sql-gen` | "DDL SQL 만들어줘" | *.sql |
| 5 | `ai-dlc-sb-migration-exec` | "Liquibase 마이그레이션 실행해줘" | changelog.xml |
| 6 | `ai-dlc-sb-layer-impl` | "레이어 구현 순서 알려줘" | (구현 가이드) |
| 7 | `ai-dlc-sb-vo-gen` | "VO 클래스 만들어줘" | *VO.java |
| 8 | `ai-dlc-sb-mapper-gen` | "Mapper 만들어줘" | *Mapper.java + XML |
| 9 | `ai-dlc-sb-service-gen` | "Service 만들어줘" | *Service.java |
| 10 | `ai-dlc-sb-controller-gen` | "Controller 만들어줘" | *Controller.java |
| 11 | `ai-dlc-sb-unit-test-gen` | "단위 테스트 만들어줘" | *Test.java |
| 12 | `ai-dlc-sb-unit-test-validate` | "테스트 코드 검증해줘" | 테스트코드_검증_*.md |
| 13 | `ai-dlc-sb-unit-test-revise` | "테스트 코드 수정해줘" | 수정된 *Test.java |
| 14 | `ai-dlc-sb-code-review` | "Spring Boot 코드 리뷰해줘" | 코드품질검토_*.md |
| 15 | `ai-dlc-sb-code-revise` | "코드 리뷰 결과 반영해줘" | 수정된 소스코드 |

#### 가이드 스킬 (참조용, 파일 생성 없음)

| 스킬 | 트리거 예시 | 내용 |
|:---|:---|:---|
| `ai-dlc-sb-springboot-guide` | "Spring Boot 사용법" | 레이어 구조, 의존성 주입 패턴 |
| `ai-dlc-sb-anyframe-guide` | "Anyframe 사용법" | Anyframe 컴포넌트 활용법 |
| `ai-dlc-sb-mybatis-guide` | "MyBatis 사용법" | Mapper XML, 동적 쿼리 패턴 |
| `ai-dlc-sb-dbio-guide` | "DBIO 사용법" | BXCM DBIO 인터페이스 패턴 |
| `ai-dlc-sb-security-guide` | "Spring Security 설정법" | JWT 인증, 인가 필터 |
| `ai-dlc-sb-db-guide` | "DB 연결 설정법" | DataSource, 트랜잭션 관리 |
| `ai-dlc-sb-liquibase-guide` | "Liquibase 사용법" | changelog 작성, 롤백 전략 |

---

### 4.2 React/Vite 프론트엔드

**적합한 경우**: SPA(Single Page Application), 별도 백엔드 API 서버와 연동

**기술 스택**: React 18 + TypeScript + Vite + Zustand + TanStack Query + Axios + shadcn/ui + Tailwind

#### 개발 흐름

```
ai-dlc-fe-project-setup       ai-dlc-fe-node-setup (선택)
(React/Vite 설정)              (Express Mock API)
        │                              │
        └──────────────┬──────────────┘
                       ▼
            ai-dlc-fe-impl-plan
            (프론트엔드구현계획_*.md)
                       │
                       ▼
            ai-dlc-fe-component-gen (반복)
            (pages/, components/, hooks/, api/)
                       │
              ┌────────┴────────────────────┐
              ▼                             ▼
   ai-dlc-fe-code-review          ai-dlc-fe-e2e-test-gen
   ai-dlc-fe-ts-check             → ai-dlc-fe-e2e-test-validate
   ai-dlc-fe-lint-check           → ai-dlc-fe-e2e-test-revise
              │
              ▼
   ai-dlc-fe-code-revise
```

#### 단계별 스킬 상세

| 순서 | 스킬 | 트리거 예시 | 산출물 |
|:---:|:---|:---|:---|
| 1 | `ai-dlc-fe-project-setup` | "React 프로젝트 만들어줘" | package.json, vite.config.ts, tsconfig.json |
| 2 | `ai-dlc-fe-node-setup` | "Mock API 서버 만들어줘" | src/app.ts, routes/ (선택) |
| 3 | `ai-dlc-fe-impl-plan` | "프론트엔드 구현 계획 세워줘" | 프론트엔드구현계획_*.md |
| 4 | `ai-dlc-fe-component-gen` | "사용자 목록 화면 만들어줘" | pages/*.tsx, hooks/*.ts, api/*.ts |
| 5 | `ai-dlc-fe-code-review` | "코드 리뷰해줘" | 코드품질검토_*.md |
| 6 | `ai-dlc-fe-ts-check` | "TypeScript 검사해줘" | TypeScript검사결과_*.md |
| 7 | `ai-dlc-fe-lint-check` | "ESLint 검사해줘" | ESLint검사결과_*.md |
| 8 | `ai-dlc-fe-code-revise` | "코드 리뷰 반영해줘" | 수정된 소스코드 |
| 9 | `ai-dlc-fe-e2e-test-gen` | "e2e 테스트 만들어줘" | tests/**/*.spec.ts |
| 10 | `ai-dlc-fe-e2e-test-validate` | "e2e 테스트 검증해줘" | e2e테스트_검증_*.md |
| 11 | `ai-dlc-fe-e2e-test-revise` | "e2e 테스트 수정해줘" | 수정된 테스트 |

#### 가이드 스킬

| 스킬 | 트리거 예시 | 내용 |
|:---|:---|:---|
| `ai-dlc-fe-shadcn-guide` | "shadcn/ui 사용법" | Button/Dialog/Form/Table 컴포넌트 패턴 |
| `ai-dlc-fe-tailwind-guide` | "Tailwind 사용법" | 반응형, flex/grid, cn() 패턴 |
| `ai-dlc-fe-axios-guide` | "Axios 사용법" | 인터셉터, 에러 처리, React Query 연동 |
| `ai-dlc-fe-zod-guide` | "Zod 사용법" | 스키마 정의, React Hook Form 연동 |
| `ai-dlc-fe-state-guide` | "Zustand 사용법" | store 생성, slice 패턴, 미들웨어 |
| `ai-dlc-fe-react-query-guide` | "React Query 사용법" | useQuery, useMutation, 낙관적 업데이트 |
| `ai-dlc-fe-perf-guide` | "React 성능 최적화" | memo, useMemo, useCallback, lazy |

---

### 4.3 Next.js App Router

**적합한 경우**: SSR/SSG/ISR이 필요한 풀스택, 인증 포함, SEO 중요

**기술 스택**: Next.js 15 + TypeScript + Auth.js v5 + TanStack Query + Zustand + Prisma + shadcn/ui + Tailwind

#### RSC vs CC 판단 기준 (가장 중요)

| 조건 | 컴포넌트 유형 |
|:---|:---:|
| 데이터 조회, SEO 필요, 정적 UI | **RSC** (기본) |
| useState, onClick, useEffect, 브라우저 API | **CC** (`'use client'`) |
| 폼 제출, 내부 뮤테이션 | **Server Actions** |
| 외부 노출 API, 파일 업로드, 웹훅 | **Route Handlers** |

#### 개발 흐름

```
ai-dlc-nxt-project-setup
(package.json, next.config.ts, Auth.js, middleware.ts)
        │
        ▼
ai-dlc-nxt-impl-plan
(RSC/CC 분류, 라우트 구조, 데이터 패칭 전략)
        │
   ┌────┴──────────────────────────────────────┐
   ▼                                           ▼
ai-dlc-nxt-page-gen              ai-dlc-nxt-route-handler-gen
(app/**/page.tsx,                (app/api/**/route.ts)
 layout.tsx, loading.tsx,                │
 error.tsx, components/)                 ▼
        │                    ai-dlc-nxt-server-action-gen
        │                    (actions/*.ts)
        └──────────────┬──────────────────┘
                       ▼
           ai-dlc-nxt-code-review (NX-001~010)
           ai-dlc-fe-ts-check
           ai-dlc-fe-lint-check
                       │
                       ▼
           ai-dlc-nxt-code-revise
                       │
                       ▼
           ai-dlc-nxt-e2e-test-gen
           → ai-dlc-fe-e2e-test-validate
           → ai-dlc-fe-e2e-test-revise
```

#### 단계별 스킬 상세

| 순서 | 스킬 | 트리거 예시 | 산출물 |
|:---:|:---|:---|:---|
| 1 | `ai-dlc-nxt-project-setup` | "Next.js 프로젝트 만들어줘" | package.json, next.config.ts, auth.ts |
| 2 | `ai-dlc-nxt-impl-plan` | "Next.js 구현 계획 세워줘" | Next.js구현계획_*.md |
| 3 | `ai-dlc-nxt-page-gen` | "사용자 목록 페이지 만들어줘" | app/**/page.tsx, components/ |
| 4 | `ai-dlc-nxt-route-handler-gen` | "GET /api/users Route Handler 만들어줘" | app/api/**/route.ts |
| 5 | `ai-dlc-nxt-server-action-gen` | "사용자 등록 Server Action 만들어줘" | actions/*.ts |
| 6 | `ai-dlc-nxt-code-review` | "Next.js 코드 리뷰해줘" | 코드품질검토_*.md |
| 7 | `ai-dlc-fe-ts-check` | "TypeScript 검사해줘" | TypeScript검사결과_*.md |
| 8 | `ai-dlc-fe-lint-check` | "ESLint 검사해줘" | ESLint검사결과_*.md |
| 9 | `ai-dlc-nxt-code-revise` | "코드 리뷰 반영해줘" | 수정된 소스코드 |
| 10 | `ai-dlc-nxt-e2e-test-gen` | "Next.js e2e 테스트 만들어줘" | tests/**/*.spec.ts |
| 11 | `ai-dlc-fe-e2e-test-validate` | "e2e 테스트 검증해줘" | e2e테스트_검증_*.md |
| 12 | `ai-dlc-fe-e2e-test-revise` | "e2e 테스트 수정해줘" | 수정된 테스트 |

#### 가이드 스킬

| 스킬 | 트리거 예시 | 내용 |
|:---|:---|:---|
| `ai-dlc-nxt-sc-guide` | "Server Component 데이터 패칭 방법" | fetch 캐싱, Suspense, cache() |
| `ai-dlc-nxt-auth-guide` | "Auth.js 로그인 구현 방법" | CredentialsProvider, 세션 접근, 역할 제어 |
| `ai-dlc-nxt-middleware-guide` | "middleware.ts 설정법" | 인증 라우트 보호, matcher 설정 |
| `ai-dlc-nxt-perf-guide` | "next/image 사용법" | 이미지 최적화, next/font, Dynamic Import |
| `ai-dlc-nxt-deploy-guide` | "Vercel 배포 방법" | Vercel, Docker standalone, ISR 전략 |
| `ai-dlc-fe-shadcn-guide` | "shadcn/ui 사용법" | React/Vite와 동일 재사용 |
| `ai-dlc-fe-tailwind-guide` | "Tailwind 사용법" | React/Vite와 동일 재사용 |
| `ai-dlc-fe-zod-guide` | "Zod 사용법" | React/Vite와 동일 재사용 |
| `ai-dlc-fe-state-guide` | "Zustand 사용법" | React/Vite와 동일 재사용 |
| `ai-dlc-fe-react-query-guide` | "React Query 사용법" | Client Component에서 재사용 |

---

### 4.4 Vue.js 3

**적합한 경우**: Vue.js SPA, Pinia 상태관리, Vue Router 기반 클라이언트 사이드 앱

**기술 스택**: Vue 3 + TypeScript + Vite + Pinia + Vue Router v4 + @tanstack/vue-query + VeeValidate v4 + Zod + shadcn-vue + Tailwind CSS + Axios

#### 개발 흐름

```
ai-dlc-vue-project-setup
        │
        ▼
ai-dlc-vue-impl-plan
        │
        ▼
ai-dlc-vue-component-gen (화면별 반복)
        │
   ┌────┴─────────────────────────────────┐
   ▼                                      ▼
ai-dlc-vue-code-review          ai-dlc-vue-e2e-test-gen
ai-dlc-vue-ts-check             → ai-dlc-fe-e2e-test-validate
ai-dlc-vue-lint-check           → ai-dlc-fe-e2e-test-revise
   │
   ▼
ai-dlc-vue-code-revise
```

#### 단계별 스킬 상세

| 순서 | 스킬 | 트리거 예시 | 산출물 |
|:---:|:---|:---|:---|
| 1 | `ai-dlc-vue-project-setup` | "Vue.js 프로젝트 만들어줘" | package.json, vite.config.ts, tsconfig.json |
| 2 | `ai-dlc-vue-impl-plan` | "Vue.js 구현 계획 세워줘" | Vue구현계획_*.md |
| 3 | `ai-dlc-vue-component-gen` | "사용자 목록 화면 만들어줘" | views/*.vue, components/*.vue, composables/*.ts |
| 4 | `ai-dlc-vue-code-review` | "Vue 코드 리뷰해줘" | 코드품질검토_*.md |
| 5 | `ai-dlc-vue-ts-check` | "TypeScript 검사해줘" | TypeScript검사결과_*.md |
| 6 | `ai-dlc-vue-lint-check` | "ESLint 검사해줘" | ESLint검사결과_*.md |
| 7 | `ai-dlc-vue-code-revise` | "코드 리뷰 반영해줘" | 수정된 소스코드 |
| 8 | `ai-dlc-vue-e2e-test-gen` | "Vue e2e 테스트 만들어줘" | tests/**/*.spec.ts |
| 9 | `ai-dlc-fe-e2e-test-validate` | "e2e 테스트 검증해줘" | e2e테스트_검증_*.md |
| 10 | `ai-dlc-fe-e2e-test-revise` | "e2e 테스트 수정해줘" | 수정된 테스트 |

#### 가이드 스킬

| 스킬 | 트리거 예시 | 내용 |
|:---|:---|:---|
| `ai-dlc-vue-pinia-guide` | "Pinia 사용법" | defineStore, storeToRefs, 액션·게터 패턴 |
| `ai-dlc-vue-router-guide` | "Vue Router 사용법" | 동적 라우트, 네비게이션 가드, useRouter |
| `ai-dlc-vue-query-guide` | "Vue Query 사용법" | useQuery, useMutation, 낙관적 업데이트 |
| `ai-dlc-vue-form-guide` | "VeeValidate 사용법" | useForm, Zod 스키마 연동, Field 컴포넌트 |
| `ai-dlc-vue-ui-guide` | "shadcn-vue 사용법" | 컴포넌트 설치, Radix-Vue 기반 패턴 |
| `ai-dlc-vue-perf-guide` | "Vue 성능 최적화" | defineAsyncComponent, v-memo, shallowRef |
| `ai-dlc-fe-tailwind-guide` | "Tailwind 사용법" | React/Vite와 동일 재사용 |
| `ai-dlc-fe-axios-guide` | "Axios 사용법" | React/Vite와 동일 재사용 |
| `ai-dlc-fe-zod-guide` | "Zod 사용법" | React/Vite와 동일 재사용 |

---

## 5. 스킬 전체 목록

### 아이디어 구체화 — Pre-Requirements (5종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-idea-clarify` | 막연한 아이디어·불편함 → 아이디어정의서 |
| `ai-dlc-persona-create` | 사용자 페르소나 정의 (PS-NNN) |
| `ai-dlc-user-story-map` | 사용자 여정 + 스토리 맵 (US-NNN) |
| `ai-dlc-mvp-scope` | MoSCoW MVP 범위 정의 + 릴리즈 로드맵 |
| `ai-dlc-idea-to-req` | 아이디어 산출물 → FR-NNN 요구사항 정의서 변환 |

### 요구사항 정의 (2종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-requirements` | 요구사항 정의서 생성 |
| `ai-dlc-service-catalog` | 서비스 카탈로그 분류 |

### 분석 (9종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-functional-req` | 기능 요구사항 정의 |
| `ai-dlc-nonfunctional-req` | 비기능 요구사항 정의 |
| `ai-dlc-biz-rules-create` | 비즈니스 규칙 도출 |
| `ai-dlc-biz-rules-validate` | 비즈니스 규칙 검증 |
| `ai-dlc-biz-rules-revise` | 비즈니스 규칙 수정 |
| `ai-dlc-glossary-create` | 도메인 용어사전 생성 |
| `ai-dlc-glossary-validate` | 용어사전 검증 |
| `ai-dlc-glossary-revise` | 용어사전 수정 |
| `ai-dlc-glossary-apply` | 요구사항에 표준 용어 반영 |

### 코드 분석 (5종, 레거시 분석)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-api-spec-extract` | 소스코드에서 OpenAPI 추출 |
| `ai-dlc-data-model-analysis` | DB 스키마 분석 |
| `ai-dlc-code-traceability` | 요구사항 추적성 분석 |
| `ai-dlc-dependency-analysis` | 의존성 분석 |
| `ai-dlc-code-complexity` | 코드 복잡도 분석 |

### 설계 (18종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-design-extract` | 레거시 설계 추출 (선택) |
| `ai-dlc-usecase-create` | 유즈케이스 생성 |
| `ai-dlc-usecase-validate` | 유즈케이스 검증 |
| `ai-dlc-usecase-revise` | 유즈케이스 수정 |
| `ai-dlc-screen-list` | 화면 목록 생성 |
| `ai-dlc-screen-spec` | 화면 설계서 생성 |
| `ai-dlc-screen-validate` | 화면 설계 검증 |
| `ai-dlc-screen-revise` | 화면 설계 수정 |
| `ai-dlc-api-design` | API 설계서 (OpenAPI) 생성 |
| `ai-dlc-api-validate` | API 설계 검증 |
| `ai-dlc-api-revise` | API 설계 수정 |
| `ai-dlc-data-design` | 데이터 설계서 생성 |
| `ai-dlc-data-validate` | 데이터 설계 검증 |
| `ai-dlc-data-revise` | 데이터 설계 수정 |
| `ai-dlc-class-design` | 클래스 설계서 생성 |
| `ai-dlc-class-validate` | 클래스 설계 검증 |
| `ai-dlc-class-revise` | 클래스 설계 수정 |
| `ai-dlc-sequence-design` | 시퀀스 다이어그램 생성 |

### 개발 — Spring Boot 백엔드 (23종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-sb-project-setup` | Spring Boot 프로젝트 초기화 |
| `ai-dlc-sb-anyframe-setup` | Anyframe 통합 설정 |
| `ai-dlc-sb-migration-plan` | DB 마이그레이션 전략 |
| `ai-dlc-sb-sql-plan` | SQL 생성 계획 |
| `ai-dlc-sb-sql-gen` | DDL/DML SQL 생성 |
| `ai-dlc-sb-migration-exec` | Liquibase/Flyway 실행 |
| `ai-dlc-sb-layer-impl` | 레이어 구현 순서 안내 |
| `ai-dlc-sb-vo-gen` | VO 클래스 생성 |
| `ai-dlc-sb-mapper-gen` | MyBatis Mapper 생성 |
| `ai-dlc-sb-service-gen` | Service 클래스 생성 |
| `ai-dlc-sb-controller-gen` | Controller 클래스 생성 |
| `ai-dlc-sb-unit-test-gen` | 단위 테스트 생성 |
| `ai-dlc-sb-unit-test-validate` | 테스트 코드 검증 |
| `ai-dlc-sb-unit-test-revise` | 테스트 코드 수정 |
| `ai-dlc-sb-code-review` | Spring Boot 코드 품질 검토 |
| `ai-dlc-sb-code-revise` | 코드 리뷰 반영 |
| `ai-dlc-sb-springboot-guide` | Spring Boot 가이드 |
| `ai-dlc-sb-anyframe-guide` | Anyframe 가이드 |
| `ai-dlc-sb-mybatis-guide` | MyBatis 가이드 |
| `ai-dlc-sb-dbio-guide` | DBIO 가이드 |
| `ai-dlc-sb-security-guide` | Spring Security 가이드 |
| `ai-dlc-sb-db-guide` | DB 연결 가이드 |
| `ai-dlc-sb-liquibase-guide` | Liquibase 가이드 |

### 개발 — React/Vite 프론트엔드 (18종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-fe-project-setup` | React/Vite 프로젝트 초기화 |
| `ai-dlc-fe-node-setup` | Node.js/Express Mock API |
| `ai-dlc-fe-impl-plan` | 프론트엔드 구현 계획 |
| `ai-dlc-fe-component-gen` | 컴포넌트·훅·API 코드 생성 |
| `ai-dlc-fe-e2e-test-gen` | Playwright e2e 테스트 생성 |
| `ai-dlc-fe-e2e-test-validate` | e2e 테스트 검증 |
| `ai-dlc-fe-e2e-test-revise` | e2e 테스트 수정 |
| `ai-dlc-fe-code-review` | React 코드 품질 검토 |
| `ai-dlc-fe-code-revise` | 코드 리뷰 반영 |
| `ai-dlc-fe-ts-check` | TypeScript 타입 검사 |
| `ai-dlc-fe-lint-check` | ESLint 코드 스타일 검사 |
| `ai-dlc-fe-shadcn-guide` | shadcn/ui 가이드 |
| `ai-dlc-fe-tailwind-guide` | Tailwind CSS 가이드 |
| `ai-dlc-fe-axios-guide` | Axios 가이드 |
| `ai-dlc-fe-zod-guide` | Zod 가이드 |
| `ai-dlc-fe-state-guide` | Zustand 가이드 |
| `ai-dlc-fe-react-query-guide` | TanStack Query 가이드 |
| `ai-dlc-fe-perf-guide` | React 성능 최적화 가이드 |

### 개발 — Next.js App Router (13종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-nxt-project-setup` | Next.js 15 프로젝트 초기화 |
| `ai-dlc-nxt-impl-plan` | Next.js 구현 전략 계획 |
| `ai-dlc-nxt-page-gen` | 페이지·레이아웃·컴포넌트 생성 |
| `ai-dlc-nxt-route-handler-gen` | Route Handlers 생성 |
| `ai-dlc-nxt-server-action-gen` | Server Actions 생성 |
| `ai-dlc-nxt-e2e-test-gen` | Playwright e2e 테스트 생성 |
| `ai-dlc-nxt-code-review` | Next.js 코드 품질 검토 |
| `ai-dlc-nxt-code-revise` | 코드 리뷰 반영 |
| `ai-dlc-nxt-sc-guide` | Server Components 가이드 |
| `ai-dlc-nxt-auth-guide` | Auth.js v5 인증 가이드 |
| `ai-dlc-nxt-middleware-guide` | Next.js 미들웨어 가이드 |
| `ai-dlc-nxt-perf-guide` | 성능 최적화 가이드 |
| `ai-dlc-nxt-deploy-guide` | 배포 가이드 |

### 개발 — Vue.js 3 (14종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-vue-project-setup` | Vue.js 3 프로젝트 초기화 |
| `ai-dlc-vue-impl-plan` | Vue.js 구현 전략 계획 |
| `ai-dlc-vue-component-gen` | SFC 컴포넌트·Composable·API 코드 생성 |
| `ai-dlc-vue-e2e-test-gen` | Playwright e2e 테스트 생성 |
| `ai-dlc-vue-code-review` | Vue.js 코드 품질 검토 |
| `ai-dlc-vue-ts-check` | vue-tsc 타입 검사 |
| `ai-dlc-vue-lint-check` | ESLint + eslint-plugin-vue 검사 |
| `ai-dlc-vue-code-revise` | 코드 리뷰 반영 |
| `ai-dlc-vue-pinia-guide` | Pinia 상태관리 가이드 |
| `ai-dlc-vue-router-guide` | Vue Router v4 가이드 |
| `ai-dlc-vue-query-guide` | @tanstack/vue-query 가이드 |
| `ai-dlc-vue-form-guide` | VeeValidate v4 + Zod 가이드 |
| `ai-dlc-vue-ui-guide` | shadcn-vue 컴포넌트 가이드 |
| `ai-dlc-vue-perf-guide` | Vue.js 성능 최적화 가이드 |

### 변경 관리 (5종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-change-register` | 변경 요청(CR) 등록 및 채번 |
| `ai-dlc-change-complete` | CR 완료 처리 및 상태 업데이트 |
| `ai-dlc-consistency-check` | 산출물 간 일관성 검증 |
| `ai-dlc-impact-analysis` | 변경 영향도 분석 |
| `ai-dlc-doc-impact` | 문서 영향도 분석 |

### 유틸리티 (2종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-program-spec` | 프로그램 명세서 생성 |
| `ai-dlc-md-to-word` | 마크다운 → Word 변환 |

---

## 6. ID 체계 및 산출물 연계

각 산출물은 고유 ID를 가지며, 후속 스킬의 입력으로 연결된다.

```
요구사항 정의서
  FR-NNN (기능 요구사항)
  PR/SR/QR/IR/DR/CR-NNN (비기능 요구사항)
        │
        ▼
서비스 카탈로그
  SC-NNN (서비스)  ← FR-NNN 참조
        │
        ▼
유즈케이스
  UC-NNN           ← FR-NNN, SC-NNN 참조
        │
   ┌────┴──────────────────────────┐
   ▼                               ▼
화면 (SCR-NNN)               API (operationId)
   ← UC-NNN 참조              ← UC-NNN, SCR-NNN 참조
        │                          │
        └────────────┬─────────────┘
                     ▼
             데이터 설계 (테이블명)  ← UC-NNN 참조
                     │
                     ▼
             클래스 설계 (CLS-NNN)  ← UC-NNN, operationId, 테이블명 참조
                     │
                     ▼
             시퀀스 (SEQ-NNN)       ← UC-NNN, CLS-NNN 참조
                     │
                     ▼
             [개발 단계 — 아키텍처별 분기]
```

### 연계 규칙

- 설계서에서 참조하는 ID는 반드시 이전 단계 산출물에서 확인
- UC-NNN이 없으면 화면설계·API설계를 시작하지 않는 것을 권장
- operationId는 API 설계서 기준, 코드 생성 시 메서드명·엔드포인트 기준이 됨

---

## 7. 산출물 파일 규칙

### 파일명 형식

```
{산출물유형}_{사업명}_{YYYYMMDD}.md
```

예시:
- `요구사항정의서_쇼핑몰프로젝트_20260523.md`
- `유즈케이스_쇼핑몰프로젝트_20260523.md`
- `openapi_쇼핑몰프로젝트_20260523.yaml`

수정(revise) 산출물은 버전 번호 추가:
- `비즈니스규칙_쇼핑몰프로젝트_20260523_v0.2.md`

### 저장 위치

기본적으로 현재 작업 디렉터리에 저장된다. 스킬 실행 후 절대 경로가 채팅창에 출력된다.

### 대화창 출력 (파일 미생성)

```
"보여만 줘", "파일 만들지 마", "여기에 출력", "대화창에", "표시만"
```

위 키워드를 포함하면 파일을 만들지 않고 채팅창에 내용을 출력한다.

---

## 8. 스킬 사용법

### 기본 사용법

```
# 스킬 실행 — 트리거 문장을 자연어로 입력
"요구사항 정의서 작성해줘"

# 특정 파일을 참조할 때 — @ 기호로 파일 첨부
"@화면설계서_20260523.md 를 보고 유즈케이스 만들어줘"

# 가이드 스킬 — 파일 없이 채팅창에 내용 출력
"Zustand 사용법 알려줘"
"Auth.js v5 로그인 구현 방법 알려줘"
```

### 검증→수정 반복 패턴

```
1. 생성 스킬 실행 (create/gen)
2. 검증 스킬 실행 (validate)
3. 이슈가 있으면 수정 스킬 실행 (revise)
4. 다시 2번으로 (기준 충족까지 반복)
```

### 코드 생성 후 품질 관리 패턴

```
1. 코드 생성 (component-gen / page-gen / controller-gen 등)
2. code-review → 이슈 코드 목록 확인
3. ts-check → TypeScript 오류 확인
4. lint-check → ESLint 규칙 위반 확인
5. code-revise → 이슈 우선순위 순서로 수정
6. e2e-test-gen → 통합 테스트 코드 생성
7. e2e-test-validate → 테스트 코드 품질 확인
8. e2e-test-revise → 테스트 코드 보완
```

### 자주 하는 질문

**Q: 어떤 스킬을 써야 할지 모를 때**
> "지금 상황에서 어떤 스킬을 사용하면 되나요?" 라고 물어보면 현재 상황에 맞는 스킬을 안내해 준다.

**Q: 기존 코드에 스킬을 적용할 때**
> `@파일명` 으로 기존 파일을 첨부하거나 "현재 프로젝트의 코드를 보고 ~해줘" 라고 입력한다.

**Q: React와 Next.js 중 어떤 걸 써야 하나요**
> SEO가 필요하거나, 서버 사이드 렌더링/인증/풀스택이 필요하면 Next.js. 단순 SPA이고 별도 API 서버가 있으면 React/Vite.

**Q: Next.js에서 `'use client'`를 언제 붙이나요**
> `useState`, `useEffect`, `onClick` 등 클라이언트 인터랙션이 필요할 때만 붙인다. 데이터만 조회하는 컴포넌트는 RSC(기본값)로 둔다.

**Q: Server Actions vs Route Handlers 차이**
> 폼 제출·내부 뮤테이션 → Server Actions. 외부 클라이언트(모바일 앱·타 서비스) API 노출·웹훅 수신 → Route Handlers.

---

> 이 문서는 `plans/` 의 PLAN 파일들과 `skills/ai-dlc-common/references/` 를 기반으로 작성되었습니다.
> 스킬 추가·변경 시 이 문서도 함께 업데이트하세요.
