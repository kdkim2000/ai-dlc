# AI-DLC (AI-Driven Development Lifecycle)

> **Claude Code 스킬을 활용하여 요구사항 정의부터 코드 생성·검증까지 전체 개발 생명주기를 AI와 함께 진행하는 방법론**

![스킬 수](https://img.shields.io/badge/스킬-110종-blue) ![아키텍처](https://img.shields.io/badge/아키텍처-4종-green) ![언어](https://img.shields.io/badge/언어-한국어-red)

---

## 목차

1. [AI-DLC란?](#1-ai-dlc란)
2. [스킬 설치 방법](#2-스킬-설치-방법)
3. [빠른 시작](#3-빠른-시작)
4. [전체 프로세스 개요](#4-전체-프로세스-개요)
5. [단계별 상세 가이드](#5-단계별-상세-가이드)
   - [5-1. 요구사항 정의](#5-1-요구사항-정의)
   - [5-2. 분석](#5-2-분석)
   - [5-3. 설계](#5-3-설계)
   - [5-4. 개발 — Spring Boot 백엔드](#5-4-개발--spring-boot-백엔드)
   - [5-5. 개발 — React/Vite 프론트엔드](#5-5-개발--reactvite-프론트엔드)
   - [5-6. 개발 — Next.js App Router](#5-6-개발--nextjs-app-router)
   - [5-7. 개발 — Vue.js 3](#5-7-개발--vuejs-3)
   - [5-8. 변경 관리](#5-8-변경-관리)
6. [스킬 전체 목록 (110종)](#6-스킬-전체-목록-110종)
7. [ID 체계 및 산출물 연계](#7-id-체계-및-산출물-연계)
8. [산출물 파일 규칙](#8-산출물-파일-규칙)
9. [스킬 사용 팁](#9-스킬-사용-팁)
10. [자주 묻는 질문 (FAQ)](#10-자주-묻는-질문-faq)
11. [이 저장소 구조](#11-이-저장소-구조)

---

## 1. AI-DLC란?

**AI-DLC(AI-Driven Development Lifecycle)** 는 Claude Code 스킬을 활용하여 소프트웨어 개발의 전 과정을 AI와 협업하는 방법론입니다.

### 왜 AI-DLC인가?

전통적인 개발 방식에서는 개발자가 직접 설계서를 작성하고, 그 설계서를 보며 코드를 작성하고, 다시 코드를 검토하고 수정하는 반복 작업을 혼자 또는 소수 팀이 수행합니다. AI-DLC는 이 과정의 각 단계를 **스킬**이라는 단위로 정형화하여 Claude Code가 일관된 산출물을 생성하도록 지원합니다.

```
기존 방식:  개발자 → [설계서 작성] → [코드 작성] → [검토/수정] → 반복
AI-DLC:    개발자 → "스킬 트리거 문장 입력" → Claude가 산출물 생성 → 검증·수정
```

### 지원 아키텍처

| 구분 | 기술 스택 | 스킬 접두사 |
|:---|:---|:---|
| Java 백엔드 REST API | Spring Boot + MyBatis + Liquibase | `ai-dlc-sb-*` |
| React SPA | React 18 + Vite + Zustand + TanStack Query | `ai-dlc-fe-*` |
| React 풀스택 | Next.js 15 App Router + Auth.js v5 | `ai-dlc-nxt-*` |
| Vue.js SPA | Vue 3 + Vite + Pinia + Vue Router v4 | `ai-dlc-vue-*` |

---

## 2. 스킬 설치 방법

이 저장소의 `skills/` 디렉터리에 있는 스킬 파일을 Claude Code의 사용자 스킬 경로에 복사하면 바로 사용할 수 있습니다.

### Windows

```powershell
# 1. 저장소를 clone
git clone https://github.com/kdkim2000/ai-dlc.git

# 2. 스킬 파일을 Claude Code 스킬 경로에 복사
#    (아래 명령을 PowerShell에서 실행)
Copy-Item -Path "ai-dlc\skills\*" `
          -Destination "$env:USERPROFILE\.claude\skills\" `
          -Recurse -Force

# 3. Claude Code 재시작 → 스킬 자동 인식
```

### macOS / Linux

```bash
# 1. 저장소를 clone
git clone https://github.com/kdkim2000/ai-dlc.git

# 2. 스킬 파일을 Claude Code 스킬 경로에 복사
cp -r ai-dlc/skills/* ~/.claude/skills/

# 3. Claude Code 재시작 → 스킬 자동 인식
```

### 설치 확인

Claude Code 채팅창에서 아래와 같이 입력하면 설치된 스킬 목록을 확인할 수 있습니다.

```
"설치된 AI-DLC 스킬 목록 보여줘"
```

또는 `/find-skills` 명령을 사용하세요.

---

## 3. 빠른 시작

### 스킬 실행 방법

스킬은 **자연어 트리거 문장**으로 실행됩니다. 별도의 명령어나 `/스킬명` 형식이 필요 없습니다.

```
Claude Code 채팅창에 자연어로 입력:

예) "요구사항 정의서 작성해줘"
    "유즈케이스 만들어줘"
    "Spring Boot 프로젝트 설정해줘"
    "Vue.js 컴포넌트 만들어줘"
    "e2e 테스트 생성해줘"
```

### 신규 프로젝트 최소 경로 (5단계)

```
1단계  "요구사항 정의서 작성해줘"        → 요구사항정의서_*.md
          ↓
2단계  "유즈케이스 만들어줘"             → 유즈케이스_*.md
          ↓
3단계  "화면 목록 만들어줘"              → 화면목록_*.md
       "API 설계해줘"                    → openapi_*.yaml
          ↓
4단계  [아키텍처 선택 후] 프로젝트 설정   → 초기 설정 파일 일체
       + 코드 생성                        → 소스코드
          ↓
5단계  "코드 리뷰해줘"                   → 코드품질검토_*.md
       "e2e 테스트 만들어줘"             → tests/**/*.spec.ts
```

### 아키텍처 선택 기준

| 상황 | 선택 |
|:---|:---|
| Java 기반 REST API 서버, 기존 Java 팀 | Spring Boot (`ai-dlc-sb-*`) |
| React SPA, 별도 API 서버와 연동 | React/Vite (`ai-dlc-fe-*`) |
| SSR/SEO 필요, 인증 포함, 풀스택 | Next.js App Router (`ai-dlc-nxt-*`) |
| Vue.js SPA, Pinia 상태관리, Vue Router | Vue.js 3 (`ai-dlc-vue-*`) |

---

## 4. 전체 프로세스 개요

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          AI-DLC 전체 흐름                                    │
│                                                                             │
│   [요구사항 정의]  →  [분석]  →  [설계]  →  [개발]  →  [검증]              │
│                                                                             │
│   요구사항 정의서       비즈니스 규칙     코드 생성     e2e 테스트           │
│   서비스 카탈로그       도메인 용어사전   단위 테스트   코드 리뷰            │
│   기능 요구사항         유즈케이스        코드 검증     품질 검토            │
│   비기능 요구사항       화면·API 설계                                       │
│                         데이터 설계                                          │
│                         클래스 설계                                          │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 개발 단계 아키텍처 분기

```
[설계 완료: UC-NNN, SCR-NNN, API YAML, 데이터설계서, 클래스설계서]
                              │
          ┌───────────────────┼──────────────────────┐
          ▼                   ▼                       ▼                  ▼
   [Spring Boot]         [React/Vite]           [Next.js]           [Vue.js 3]
   ai-dlc-sb-*           ai-dlc-fe-*           ai-dlc-nxt-*        ai-dlc-vue-*
```

### 변경 관리 (언제든지 적용 가능)

```
개발 진행 중 변경 발생
    │
    ▼
ai-dlc-change-register   →   ai-dlc-impact-analysis   →   ai-dlc-doc-impact
(CR 등록)                    (영향 범위 분석)              (수정 문서 식별)
    │
    ▼
[코드·설계서 수정 진행]
    │
    ▼
ai-dlc-consistency-check  →  ai-dlc-change-complete
(일관성 검증)                (완료 처리)
```

---

## 5. 단계별 상세 가이드

### 5-1. 요구사항 정의

**목표**: 기획서·인터뷰·아이디어를 정형화된 요구사항 문서로 변환

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

> **팁**: `@기획서.md` 처럼 @ 기호로 기존 파일을 첨부하면 해당 내용을 기반으로 요구사항을 작성해 줍니다.

---

### 5-2. 분석

**목표**: 요구사항에서 비즈니스 규칙·도메인 용어를 도출하고 표준화

```
입력: 요구사항 정의서, 서비스 카탈로그
출력: 비즈니스 규칙 테이블(BR-NNN), 도메인 표준 용어사전(TM-NNN)
```

#### 비즈니스 규칙·용어사전

| 순서 | 스킬 | 트리거 예시 | 산출물 |
|:---:|:---|:---|:---|
| 1 | `ai-dlc-biz-rules-create` | "비즈니스 규칙 도출해줘" | 비즈니스규칙_*.md |
| 2 | `ai-dlc-biz-rules-validate` | "비즈니스 규칙 검증해줘" | 검증 보고서 |
| 3 | `ai-dlc-biz-rules-revise` | "비즈니스 규칙 수정해줘" | 비즈니스규칙_*_v0.2.md |
| 4 | `ai-dlc-glossary-create` | "도메인 용어사전 만들어줘" | 용어사전_*.md |
| 5 | `ai-dlc-glossary-validate` | "용어사전 검증해줘" | 검증 보고서 |
| 6 | `ai-dlc-glossary-revise` | "용어사전 수정해줘" | 용어사전_*_v0.2.md |
| 7 | `ai-dlc-glossary-apply` | "요구사항에 표준 용어 반영해줘" | 요구사항정의서_*_v2.md |

#### 코드 분석 (기존 시스템이 있을 때)

레거시 코드나 기존 시스템이 있을 경우, 코드에서 직접 설계 정보를 추출합니다.

| 스킬 | 트리거 예시 | 산출물 |
|:---|:---|:---|
| `ai-dlc-api-spec-extract` | "소스코드에서 API 명세 추출해줘" | OpenAPI YAML |
| `ai-dlc-data-model-analysis` | "DB 스키마 분석해줘" | 데이터모델분석_*.md |
| `ai-dlc-code-traceability` | "코드 추적성 분석해줘" | 추적성분석_*.md |
| `ai-dlc-dependency-analysis` | "의존성 분석해줘" | 의존성분석_*.md |
| `ai-dlc-code-complexity` | "코드 복잡도 분석해줘" | 복잡도분석_*.md |
| `ai-dlc-design-extract` | "레거시 설계 추출해줘" | 설계기초추출서_*.md |

---

### 5-3. 설계

**목표**: 유즈케이스 → 화면·API·데이터·클래스 설계서 생성

```
입력: 요구사항(FR-NNN), 서비스 카탈로그(SC-NNN), 도메인 용어사전
출력: UC-NNN, SCR-NNN, API YAML, 데이터설계서, 클래스설계서, 시퀀스다이어그램
```

#### 설계 흐름

```
ai-dlc-usecase-create (UC-NNN)
        │
   ┌────┴──────────────────────────────────────┐
   ▼                                           ▼
ai-dlc-screen-list (SCR 목록)          ai-dlc-api-design (OpenAPI YAML)
ai-dlc-screen-spec (화면 상세)         ai-dlc-api-validate
ai-dlc-screen-validate                 ai-dlc-api-revise
ai-dlc-screen-revise                   (validate→revise 반복)
        │                                      │
        └──────────────┬───────────────────────┘
                       ▼
              ai-dlc-data-design → ai-dlc-data-validate → ai-dlc-data-revise
                       │
                       ▼
              ai-dlc-class-design → ai-dlc-class-validate → ai-dlc-class-revise
                       │
                       ▼
              ai-dlc-sequence-design (필요한 유즈케이스만 선택)
```

#### 단계별 스킬 상세

| 순서 | 스킬 | 트리거 예시 | 산출물 |
|:---:|:---|:---|:---|
| 1 | `ai-dlc-usecase-create` | "유즈케이스 만들어줘" | 유즈케이스_*.md |
| 2 | `ai-dlc-usecase-validate` | "유즈케이스 검증해줘" | 검증 보고서 |
| 3 | `ai-dlc-usecase-revise` | "유즈케이스 수정해줘" | 유즈케이스_*_v0.2.md |
| 4 | `ai-dlc-screen-list` | "화면 목록 만들어줘" | 화면목록_*.md |
| 5 | `ai-dlc-screen-spec` | "화면설계서 만들어줘" | 화면설계서_*.md |
| 6 | `ai-dlc-screen-validate` | "화면설계 검증해줘" | 검증 보고서 |
| 7 | `ai-dlc-screen-revise` | "화면설계 수정해줘" | 화면설계서_*_v0.2.md |
| 8 | `ai-dlc-api-design` | "API 설계해줘" | openapi_*.yaml |
| 9 | `ai-dlc-api-validate` | "API 설계 검증해줘" | 검증 보고서 |
| 10 | `ai-dlc-api-revise` | "API 설계 수정해줘" | openapi_*_v0.2.yaml |
| 11 | `ai-dlc-data-design` | "데이터 설계해줘" | 데이터설계서_*.md |
| 12 | `ai-dlc-data-validate` | "데이터 설계 검증해줘" | 검증 보고서 |
| 13 | `ai-dlc-data-revise` | "데이터 설계 수정해줘" | 데이터설계서_*_v0.2.md |
| 14 | `ai-dlc-class-design` | "클래스 설계해줘" | 클래스설계서_*.md |
| 15 | `ai-dlc-class-validate` | "클래스 설계 검증해줘" | 검증 보고서 |
| 16 | `ai-dlc-class-revise` | "클래스 설계 수정해줘" | 클래스설계서_*_v0.2.md |
| 17 | `ai-dlc-sequence-design` | "시퀀스 다이어그램 만들어줘" | 시퀀스다이어그램_*.md |

---

### 5-4. 개발 — Spring Boot 백엔드

**적합한 경우**: Java 기반 REST API 서버, Anyframe 프레임워크 활용

**기술 스택**: Spring Boot + Java + MyBatis + Liquibase/Flyway + PostgreSQL/MySQL

#### 개발 흐름

```
ai-dlc-sb-project-setup (또는 ai-dlc-sb-anyframe-setup)
        │
   ┌────┴──────────────────────────────────────┐
   ▼                                           ▼
ai-dlc-sb-migration-plan              ai-dlc-sb-layer-impl
ai-dlc-sb-sql-plan                    (구현 순서 안내)
ai-dlc-sb-sql-gen                             │
ai-dlc-sb-migration-exec          ┌───────────┼───────────┐
(DB 스키마 생성)                   ▼           ▼           ▼
                             vo-gen     mapper-gen   (다음 단계)
                                              │
                                              ▼
                                        service-gen
                                              │
                                              ▼
                                       controller-gen
                                              │
                               ┌──────────────┴──────────────┐
                               ▼                             ▼
                    ai-dlc-sb-unit-test-gen       ai-dlc-sb-code-review
                    → unit-test-validate          → ai-dlc-sb-code-revise
                    → unit-test-revise
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
| 6 | `ai-dlc-sb-layer-impl` | "레이어 구현 순서 알려줘" | 구현 가이드 |
| 7 | `ai-dlc-sb-vo-gen` | "VO 클래스 만들어줘" | *VO.java |
| 8 | `ai-dlc-sb-mapper-gen` | "Mapper 만들어줘" | *Mapper.java + XML |
| 9 | `ai-dlc-sb-service-gen` | "Service 만들어줘" | *Service.java + Impl |
| 10 | `ai-dlc-sb-controller-gen` | "Controller 만들어줘" | *Controller.java |
| 11 | `ai-dlc-sb-unit-test-gen` | "단위 테스트 만들어줘" | *Test.java |
| 12 | `ai-dlc-sb-unit-test-validate` | "테스트 코드 검증해줘" | 테스트코드검증_*.md |
| 13 | `ai-dlc-sb-unit-test-revise` | "테스트 코드 수정해줘" | 수정된 *Test.java |
| 14 | `ai-dlc-sb-code-review` | "Spring Boot 코드 리뷰해줘" | 코드품질검토_*.md |
| 15 | `ai-dlc-sb-code-revise` | "코드 리뷰 결과 반영해줘" | 수정된 소스코드 |

#### 가이드 스킬 (파일 생성 없이 내용 안내)

| 스킬 | 트리거 예시 | 안내 내용 |
|:---|:---|:---|
| `ai-dlc-sb-springboot-guide` | "Spring Boot 사용법 알려줘" | 레이어 구조, 의존성 주입 패턴 |
| `ai-dlc-sb-anyframe-guide` | "Anyframe 사용법 알려줘" | Anyframe 컴포넌트 활용법 |
| `ai-dlc-sb-mybatis-guide` | "MyBatis 사용법 알려줘" | Mapper XML, 동적 쿼리 패턴 |
| `ai-dlc-sb-dbio-guide` | "DBIO 사용법 알려줘" | BXCM DBIO 인터페이스 패턴 |
| `ai-dlc-sb-security-guide` | "Spring Security 설정법 알려줘" | JWT 인증, 인가 필터 구성 |
| `ai-dlc-sb-db-guide` | "DB 연결 설정법 알려줘" | DataSource, 트랜잭션 관리 |
| `ai-dlc-sb-liquibase-guide` | "Liquibase 사용법 알려줘" | changelog 작성, 롤백 전략 |

---

### 5-5. 개발 — React/Vite 프론트엔드

**적합한 경우**: SPA(Single Page Application), 별도 백엔드 API 서버와 연동

**기술 스택**: React 18 + TypeScript + Vite + Zustand + TanStack Query v5 + React Hook Form + Zod + shadcn/ui + Tailwind CSS + Axios

#### 개발 흐름

```
ai-dlc-fe-project-setup         ai-dlc-fe-node-setup (선택)
(React/Vite 초기 설정)           (Express Mock API)
        │                                │
        └──────────────┬────────────────┘
                       ▼
            ai-dlc-fe-impl-plan
            (프론트엔드구현계획_*.md)
                       │
                       ▼
            ai-dlc-fe-component-gen (화면별 반복)
            (pages/, components/, hooks/, api/)
                       │
              ┌────────┴────────────────────────┐
              ▼                                 ▼
   ai-dlc-fe-code-review            ai-dlc-fe-e2e-test-gen
   ai-dlc-fe-ts-check               → ai-dlc-fe-e2e-test-validate
   ai-dlc-fe-lint-check             → ai-dlc-fe-e2e-test-revise
              │
              ▼
   ai-dlc-fe-code-revise
```

#### 단계별 스킬 상세

| 순서 | 스킬 | 트리거 예시 | 산출물 |
|:---:|:---|:---|:---|
| 1 | `ai-dlc-fe-project-setup` | "React 프로젝트 만들어줘" | package.json, vite.config.ts, tsconfig.json 등 |
| 2 | `ai-dlc-fe-node-setup` | "Mock API 서버 만들어줘" | src/app.ts, routes/ (선택) |
| 3 | `ai-dlc-fe-impl-plan` | "프론트엔드 구현 계획 세워줘" | 프론트엔드구현계획_*.md |
| 4 | `ai-dlc-fe-component-gen` | "사용자 목록 화면 만들어줘" | pages/*.tsx, hooks/*.ts, api/*.ts |
| 5 | `ai-dlc-fe-code-review` | "React 코드 리뷰해줘" | 코드품질검토_*.md |
| 6 | `ai-dlc-fe-ts-check` | "TypeScript 검사해줘" | TypeScript검사결과_*.md |
| 7 | `ai-dlc-fe-lint-check` | "ESLint 검사해줘" | ESLint검사결과_*.md |
| 8 | `ai-dlc-fe-code-revise` | "코드 리뷰 결과 반영해줘" | 수정된 소스코드 |
| 9 | `ai-dlc-fe-e2e-test-gen` | "e2e 테스트 만들어줘" | tests/**/*.spec.ts |
| 10 | `ai-dlc-fe-e2e-test-validate` | "e2e 테스트 검증해줘" | e2e테스트검증_*.md |
| 11 | `ai-dlc-fe-e2e-test-revise` | "e2e 테스트 수정해줘" | 수정된 테스트 |

#### 가이드 스킬

| 스킬 | 트리거 예시 | 안내 내용 |
|:---|:---|:---|
| `ai-dlc-fe-shadcn-guide` | "shadcn/ui 사용법 알려줘" | Button/Dialog/Form/Table 컴포넌트 패턴 |
| `ai-dlc-fe-tailwind-guide` | "Tailwind 사용법 알려줘" | 반응형, flex/grid, cn() 패턴 |
| `ai-dlc-fe-axios-guide` | "Axios 사용법 알려줘" | 인터셉터, 에러 처리, React Query 연동 |
| `ai-dlc-fe-zod-guide` | "Zod 스키마 사용법 알려줘" | 스키마 정의, React Hook Form 연동 |
| `ai-dlc-fe-state-guide` | "Zustand 사용법 알려줘" | store 생성, slice 패턴, 미들웨어 |
| `ai-dlc-fe-react-query-guide` | "React Query 사용법 알려줘" | useQuery, useMutation, 낙관적 업데이트 |
| `ai-dlc-fe-perf-guide` | "React 성능 최적화 방법 알려줘" | memo, useMemo, useCallback, lazy 로딩 |

---

### 5-6. 개발 — Next.js App Router

**적합한 경우**: SSR/SEO 필요, 인증 포함, 풀스택 단일 프레임워크

**기술 스택**: Next.js 15 + TypeScript + Auth.js v5 + TanStack Query v5 + Zustand + shadcn/ui + Tailwind CSS + Prisma (선택)

#### RSC vs CC 판단 기준

| 조건 | 컴포넌트 유형 |
|:---|:---:|
| 데이터 조회, SEO 필요, 정적 UI | **RSC** (기본값, `'use server'` 불필요) |
| `useState`, `onClick`, `useEffect`, 브라우저 API 사용 | **CC** (`'use client'` 추가) |
| 폼 제출, DB 직접 뮤테이션 | **Server Actions** (`'use server'`) |
| 외부 노출 API, 파일 업로드, 웹훅 수신 | **Route Handlers** (`app/api/**/route.ts`) |

#### 개발 흐름

```
ai-dlc-nxt-project-setup
(package.json, next.config.ts, Auth.js, middleware.ts)
        │
        ▼
ai-dlc-nxt-impl-plan
(RSC/CC 분류, 라우트 구조, 데이터 패칭 전략)
        │
   ┌────┴────────────────────────────────────────┐
   ▼                                             ▼
ai-dlc-nxt-page-gen                  ai-dlc-nxt-route-handler-gen
(app/**/page.tsx, layout.tsx,        (app/api/**/route.ts)
 loading.tsx, error.tsx)                         │
        │                            ai-dlc-nxt-server-action-gen
        └────────────┬───────────────────────────┘
                     ▼
          ai-dlc-nxt-code-review (NX 이슈 코드)
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
| 1 | `ai-dlc-nxt-project-setup` | "Next.js 프로젝트 만들어줘" | package.json, next.config.ts, auth.ts 등 |
| 2 | `ai-dlc-nxt-impl-plan` | "Next.js 구현 계획 세워줘" | Next.js구현계획_*.md |
| 3 | `ai-dlc-nxt-page-gen` | "사용자 목록 페이지 만들어줘" | app/**/page.tsx, components/ |
| 4 | `ai-dlc-nxt-route-handler-gen` | "GET /api/users Route Handler 만들어줘" | app/api/**/route.ts |
| 5 | `ai-dlc-nxt-server-action-gen` | "사용자 등록 Server Action 만들어줘" | actions/*.ts |
| 6 | `ai-dlc-nxt-code-review` | "Next.js 코드 리뷰해줘" | 코드품질검토_*.md |
| 7 | `ai-dlc-fe-ts-check` | "TypeScript 검사해줘" | TypeScript검사결과_*.md |
| 8 | `ai-dlc-fe-lint-check` | "ESLint 검사해줘" | ESLint검사결과_*.md |
| 9 | `ai-dlc-nxt-code-revise` | "Next.js 코드 리뷰 결과 반영해줘" | 수정된 소스코드 |
| 10 | `ai-dlc-nxt-e2e-test-gen` | "Next.js e2e 테스트 만들어줘" | tests/**/*.spec.ts |
| 11 | `ai-dlc-fe-e2e-test-validate` | "e2e 테스트 검증해줘" | e2e테스트검증_*.md |
| 12 | `ai-dlc-fe-e2e-test-revise` | "e2e 테스트 수정해줘" | 수정된 테스트 |

#### 가이드 스킬

| 스킬 | 트리거 예시 | 안내 내용 |
|:---|:---|:---|
| `ai-dlc-nxt-sc-guide` | "Server Component 데이터 패칭 방법 알려줘" | fetch 캐싱, Suspense, cache() 함수 |
| `ai-dlc-nxt-auth-guide` | "Auth.js 로그인 구현 방법 알려줘" | CredentialsProvider, 세션 접근, 역할 제어 |
| `ai-dlc-nxt-middleware-guide` | "middleware.ts 설정법 알려줘" | 인증 라우트 보호, matcher 설정 |
| `ai-dlc-nxt-perf-guide` | "Next.js 성능 최적화 방법 알려줘" | next/image, next/font, Dynamic Import |
| `ai-dlc-nxt-deploy-guide` | "Next.js 배포 방법 알려줘" | Vercel, Docker standalone, ISR 전략 |

---

### 5-7. 개발 — Vue.js 3

**적합한 경우**: Vue.js SPA, Pinia 상태관리, Vue Router 기반 클라이언트 사이드 앱

**기술 스택**: Vue 3 + TypeScript + Vite + Pinia + Vue Router v4 + @tanstack/vue-query + VeeValidate v4 + Zod + shadcn-vue + Tailwind CSS + Axios

#### React vs Vue.js 선택 기준

| 상황 | 선택 |
|:---|:---:|
| 팀이 React 생태계에 익숙 | React/Vite |
| 팀이 Vue.js 생태계에 익숙 | Vue.js 3 |
| Options API 기반 레거시 Vue 프로젝트 마이그레이션 | Vue.js 3 (Composition API로 전환) |
| SEO, SSR, 풀스택 | Next.js (React 기반) |

#### Vue.js 3 핵심 개념

| Vue.js | React 대응 | 설명 |
|:---|:---|:---|
| `<script setup lang="ts">` | `function Component()` | SFC 문법, JSX 없음 |
| `ref()`, `reactive()`, `computed()` | `useState`, `useMemo` | Composition API |
| Composable (`useXxx.ts`) | Custom Hook | 로직 재사용 단위 |
| Pinia (`defineStore`) | Zustand | 공식 전역 상태관리 |
| `storeToRefs()` | 구조분해 할당 | Pinia 반응성 보존 필수 |
| `v-for` + `:key` | `array.map()` + `key` | 리스트 렌더링 |

#### 개발 흐름

```
ai-dlc-vue-project-setup
(vite.config.ts, tsconfig.json, Pinia, Vue Router, shadcn-vue, Axios)
        │
        ▼
ai-dlc-vue-impl-plan
(Vue구현계획_*.md — 라우트 표, 컴포넌트 분류, Pinia 스토어 목록)
        │
        ▼
ai-dlc-vue-component-gen (화면별 반복)
(src/views/*.vue, src/components/*.vue,
 src/composables/useXxx.ts, src/stores/*.ts)
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
| 1 | `ai-dlc-vue-project-setup` | "Vue.js 프로젝트 만들어줘" | package.json, vite.config.ts, tsconfig.json, main.ts, router/index.ts, stores/auth.ts 등 |
| 2 | `ai-dlc-vue-impl-plan` | "Vue 구현 계획 세워줘" | Vue구현계획_*.md |
| 3 | `ai-dlc-vue-component-gen` | "사용자 목록 Vue 컴포넌트 만들어줘" | src/views/*.vue, src/components/*.vue, src/composables/*.ts, src/stores/*.ts |
| 4 | `ai-dlc-vue-code-review` | "Vue 코드 리뷰해줘" | 코드품질검토_*.md |
| 5 | `ai-dlc-vue-ts-check` | "vue-tsc TypeScript 검사해줘" | TypeScript검사결과_*.md |
| 6 | `ai-dlc-vue-lint-check` | "Vue ESLint 검사 결과 정리해줘" | ESLint검사결과_*.md |
| 7 | `ai-dlc-vue-code-revise` | "Vue 코드 리뷰 결과 반영해줘" | 수정된 소스코드 |
| 8 | `ai-dlc-vue-e2e-test-gen` | "Vue Playwright 테스트 생성해줘" | tests/**/*.spec.ts |
| 9 | `ai-dlc-fe-e2e-test-validate` | "e2e 테스트 검증해줘" | e2e테스트검증_*.md |
| 10 | `ai-dlc-fe-e2e-test-revise` | "e2e 테스트 수정해줘" | 수정된 테스트 |

#### Vue.js 코드 리뷰 이슈 코드 (VV-001~010)

| 코드 | 설명 | 심각도 |
|:---|:---|:---:|
| VV-001 | Options API 사용 — `<script setup>` 전환 권장 | 중간 |
| VV-002 | `<script setup>` 미사용, 장황한 `setup()` 반환 | 중간 |
| VV-003 | Pinia 없이 컴포넌트 간 상태 공유 (props drilling 2단계 초과) | **높음** |
| VV-004 | 복잡한 로직이 컴포넌트 내 직접 작성 — Composable 미추출 | 중간 |
| VV-005 | `defineProps`/`defineEmits` 타입 미정의 | 중간 |
| VV-006 | `watch` 대신 `watchEffect` 남용 (의존성 불명확) | 낮음 |
| VV-007 | `$router`/`$route` 직접 접근 — `useRouter()`/`useRoute()` 미사용 | 낮음 |
| VV-008 | `v-for`에 `:key` 미설정 또는 index를 key로 사용 | **높음** |
| VV-009 | 컴포넌트에서 API 직접 호출 — Vue Query/API 모듈 미사용 | **높음** |
| VV-010 | 폼 검증 없이 직접 submit — VeeValidate 미사용 | **높음** |

#### 가이드 스킬

| 스킬 | 트리거 예시 | 안내 내용 |
|:---|:---|:---|
| `ai-dlc-vue-pinia-guide` | "Pinia 사용법 알려줘" | defineStore, storeToRefs, persist 설정 |
| `ai-dlc-vue-router-guide` | "Vue Router 네비게이션 가드 설정법 알려줘" | createRouter, beforeEach, RouteMeta 타입 확장 |
| `ai-dlc-vue-query-guide` | "Vue Query useQuery 사용법 알려줘" | useQuery, useMutation, queryKey 계층 패턴 |
| `ai-dlc-vue-form-guide` | "VeeValidate Zod 폼 검증 방법 알려줘" | useForm, toTypedSchema, handleSubmit |
| `ai-dlc-vue-ui-guide` | "shadcn-vue 컴포넌트 설치 방법 알려줘" | shadcn-vue init, Button/Dialog/Table 패턴 |
| `ai-dlc-vue-perf-guide` | "Vue 성능 최적화 방법 알려줘" | defineAsyncComponent, v-memo, KeepAlive, shallowRef |
| `ai-dlc-fe-tailwind-guide` | "Tailwind 사용법 알려줘" | React/Vite와 동일하게 재사용 |
| `ai-dlc-fe-axios-guide` | "Axios 인터셉터 설정법 알려줘" | React/Vite와 동일하게 재사용 |
| `ai-dlc-fe-zod-guide` | "Zod 스키마 사용법 알려줘" | React/Vite와 동일하게 재사용 |

---

### 5-8. 변경 관리

**목표**: 개발 중 발생하는 변경 요청을 체계적으로 등록·추적·완료 처리

변경 관리 스킬은 **개발 진행 중 언제든지** 사용할 수 있습니다. 요구사항 변경, 버그 수정, 기능 추가 등 모든 변경사항을 구조화된 이력으로 관리합니다.

#### 변경 관리 흐름

```
[변경 요청 발생]
        │
        ▼
ai-dlc-change-register          CR-NNN 자동 채번
(변경 요청 등록)                docs/change-log.md에 이력 등록
        │
        ▼
ai-dlc-impact-analysis          변경이 미치는 산출물·코드 범위 분석
(영향도 분석)                   수정 필요 항목 목록 생성
        │
        ▼
ai-dlc-doc-impact               변경 후 수정 필요한 문서 식별
(문서 영향도 파악)
        │
        ▼
[코드·설계서 수정 진행]
        │
        ▼
ai-dlc-consistency-check        산출물 간 ID 불일치 탐지
(일관성 검사)                   FR-NNN ↔ UC-NNN ↔ SCR-NNN ↔ operationId 연계 검증
        │
        ▼
ai-dlc-change-complete          CR 상태를 '완료'로 업데이트
(변경 완료 처리)                change-log.md 이력 갱신
```

#### 변경 관리 스킬 상세

| 스킬 | 트리거 예시 | 역할 |
|:---|:---|:---|
| `ai-dlc-change-register` | "변경 요청 등록해줘", "CR 만들어줘", "버그 등록해줘" | 자연어 변경 요청 → CR-NNN 자동 채번 + change-log.md 등록 |
| `ai-dlc-impact-analysis` | "영향도 분석해줘", "변경 범위 파악해줘" | 변경이 미치는 산출물·코드 범위 분석 |
| `ai-dlc-doc-impact` | "문서 영향도 파악해줘", "어떤 문서 수정해야 해?" | 변경 후 수정 필요한 문서 식별 |
| `ai-dlc-consistency-check` | "일관성 검사해줘", "ID 연계 확인해줘" | 산출물 간 ID 불일치 탐지 (FR↔UC↔SCR↔API) |
| `ai-dlc-change-complete` | "변경 완료 처리해줘", "CR 닫아줘" | CR 상태 → 완료 업데이트 + change-log.md 갱신 |

---

## 6. 스킬 전체 목록 (110종)

### 요구사항 정의 (2종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-requirements` | 요구사항 정의서 생성 (FR/PR/SR/QR/IR/DR/CR-NNN) |
| `ai-dlc-service-catalog` | 서비스 카탈로그 분류 (SC-NNN) |

### 분석 (9종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-functional-req` | 기능 요구사항 정의 (FR-NNN 테이블) |
| `ai-dlc-nonfunctional-req` | 비기능 요구사항 정의 (PR/SR/QR/IR/DR/CR) |
| `ai-dlc-biz-rules-create` | 비즈니스 규칙 도출 (BR-NNN) |
| `ai-dlc-biz-rules-validate` | 비즈니스 규칙 검증 |
| `ai-dlc-biz-rules-revise` | 비즈니스 규칙 수정 |
| `ai-dlc-glossary-create` | 도메인 용어사전 생성 (TM-NNN) |
| `ai-dlc-glossary-validate` | 용어사전 검증 |
| `ai-dlc-glossary-revise` | 용어사전 수정 |
| `ai-dlc-glossary-apply` | 요구사항에 표준 용어 반영 |

### 코드 분석 (5종, 레거시 시스템 분석용)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-api-spec-extract` | 소스코드에서 OpenAPI 명세 추출 |
| `ai-dlc-data-model-analysis` | DB 스키마 분석 |
| `ai-dlc-code-traceability` | 요구사항 추적성 분석 |
| `ai-dlc-dependency-analysis` | 모듈 의존성 분석 |
| `ai-dlc-code-complexity` | 코드 복잡도 분석 |

### 설계 (18종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-design-extract` | 레거시 설계 추출 (역공학) |
| `ai-dlc-usecase-create` | 유즈케이스 시나리오 생성 (UC-NNN) |
| `ai-dlc-usecase-validate` | 유즈케이스 검증 |
| `ai-dlc-usecase-revise` | 유즈케이스 수정 |
| `ai-dlc-screen-list` | 화면 목록 생성 (SCR-NNN) |
| `ai-dlc-screen-spec` | 화면 설계서 상세 생성 |
| `ai-dlc-screen-validate` | 화면 설계 검증 |
| `ai-dlc-screen-revise` | 화면 설계 수정 |
| `ai-dlc-api-design` | API 설계서 생성 (OpenAPI 3.0 YAML) |
| `ai-dlc-api-validate` | API 설계 검증 |
| `ai-dlc-api-revise` | API 설계 수정 |
| `ai-dlc-data-design` | 데이터 설계서 생성 (ERD + DDL) |
| `ai-dlc-data-validate` | 데이터 설계 검증 |
| `ai-dlc-data-revise` | 데이터 설계 수정 |
| `ai-dlc-class-design` | 클래스 설계서 생성 (CLS-NNN) |
| `ai-dlc-class-validate` | 클래스 설계 검증 |
| `ai-dlc-class-revise` | 클래스 설계 수정 |
| `ai-dlc-sequence-design` | 시퀀스 다이어그램 생성 (SEQ-NNN) |

### 변경 관리 (5종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-change-register` | 변경 요청 등록 (CR-NNN 채번) |
| `ai-dlc-change-complete` | 변경 완료 처리 |
| `ai-dlc-consistency-check` | 산출물 간 ID 일관성 검사 |
| `ai-dlc-impact-analysis` | 변경 영향도 분석 |
| `ai-dlc-doc-impact` | 문서 영향도 파악 |

### 개발 — Spring Boot 백엔드 (23종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-sb-project-setup` | Spring Boot 프로젝트 초기화 |
| `ai-dlc-sb-anyframe-setup` | Anyframe 통합 설정 |
| `ai-dlc-sb-migration-plan` | DB 마이그레이션 전략 수립 |
| `ai-dlc-sb-sql-plan` | SQL 생성 계획 |
| `ai-dlc-sb-sql-gen` | DDL/DML SQL 생성 |
| `ai-dlc-sb-migration-exec` | Liquibase/Flyway 실행 |
| `ai-dlc-sb-layer-impl` | 레이어 구현 순서 안내 |
| `ai-dlc-sb-vo-gen` | VO 클래스 생성 |
| `ai-dlc-sb-mapper-gen` | MyBatis Mapper 생성 |
| `ai-dlc-sb-service-gen` | Service 클래스 생성 |
| `ai-dlc-sb-controller-gen` | Controller 클래스 생성 |
| `ai-dlc-sb-unit-test-gen` | JUnit 단위 테스트 생성 |
| `ai-dlc-sb-unit-test-validate` | 테스트 코드 검증 |
| `ai-dlc-sb-unit-test-revise` | 테스트 코드 수정 |
| `ai-dlc-sb-code-review` | Spring Boot 코드 품질 검토 |
| `ai-dlc-sb-code-revise` | 코드 리뷰 반영 |
| `ai-dlc-sb-springboot-guide` | Spring Boot 사용 가이드 |
| `ai-dlc-sb-anyframe-guide` | Anyframe 사용 가이드 |
| `ai-dlc-sb-mybatis-guide` | MyBatis 사용 가이드 |
| `ai-dlc-sb-dbio-guide` | DBIO 인터페이스 가이드 |
| `ai-dlc-sb-security-guide` | Spring Security 가이드 |
| `ai-dlc-sb-db-guide` | DB 연결 설정 가이드 |
| `ai-dlc-sb-liquibase-guide` | Liquibase 사용 가이드 |

### 개발 — React/Vite 프론트엔드 (18종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-fe-project-setup` | React/Vite 프로젝트 초기화 |
| `ai-dlc-fe-node-setup` | Node.js/Express Mock API 서버 |
| `ai-dlc-fe-impl-plan` | 프론트엔드 구현 계획 |
| `ai-dlc-fe-component-gen` | 컴포넌트·훅·API 코드 생성 |
| `ai-dlc-fe-e2e-test-gen` | Playwright e2e 테스트 생성 |
| `ai-dlc-fe-e2e-test-validate` | e2e 테스트 검증 |
| `ai-dlc-fe-e2e-test-revise` | e2e 테스트 수정 |
| `ai-dlc-fe-code-review` | React 코드 품질 검토 |
| `ai-dlc-fe-code-revise` | 코드 리뷰 반영 |
| `ai-dlc-fe-ts-check` | TypeScript 타입 검사 |
| `ai-dlc-fe-lint-check` | ESLint 코드 스타일 검사 |
| `ai-dlc-fe-shadcn-guide` | shadcn/ui 사용 가이드 |
| `ai-dlc-fe-tailwind-guide` | Tailwind CSS 사용 가이드 |
| `ai-dlc-fe-axios-guide` | Axios HTTP 클라이언트 가이드 |
| `ai-dlc-fe-zod-guide` | Zod 스키마 검증 가이드 |
| `ai-dlc-fe-state-guide` | Zustand 상태관리 가이드 |
| `ai-dlc-fe-react-query-guide` | TanStack Query 사용 가이드 |
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
| `ai-dlc-nxt-code-review` | Next.js 코드 품질 검토 (NX 이슈 코드) |
| `ai-dlc-nxt-code-revise` | 코드 리뷰 반영 |
| `ai-dlc-nxt-sc-guide` | Server Components 데이터 패칭 가이드 |
| `ai-dlc-nxt-auth-guide` | Auth.js v5 인증 가이드 |
| `ai-dlc-nxt-middleware-guide` | Next.js 미들웨어 가이드 |
| `ai-dlc-nxt-perf-guide` | 성능 최적화 가이드 |
| `ai-dlc-nxt-deploy-guide` | 배포 가이드 (Vercel/Docker) |

### 개발 — Vue.js 3 (14종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-vue-project-setup` | Vue 3 + Vite 프로젝트 초기화 |
| `ai-dlc-vue-impl-plan` | Vue.js 구현 전략 계획 |
| `ai-dlc-vue-component-gen` | SFC 컴포넌트·Composable 생성 |
| `ai-dlc-vue-e2e-test-gen` | Playwright e2e 테스트 생성 |
| `ai-dlc-vue-code-review` | Vue.js 코드 품질 검토 (VV 이슈 코드) |
| `ai-dlc-vue-ts-check` | vue-tsc TypeScript 검사 |
| `ai-dlc-vue-lint-check` | eslint-plugin-vue 검사 |
| `ai-dlc-vue-code-revise` | 코드 리뷰 반영 |
| `ai-dlc-vue-pinia-guide` | Pinia 상태관리 가이드 |
| `ai-dlc-vue-router-guide` | Vue Router v4 가이드 |
| `ai-dlc-vue-query-guide` | @tanstack/vue-query 가이드 |
| `ai-dlc-vue-form-guide` | VeeValidate v4 + Zod 폼 검증 가이드 |
| `ai-dlc-vue-ui-guide` | shadcn-vue / radix-vue UI 가이드 |
| `ai-dlc-vue-perf-guide` | Vue.js 성능 최적화 가이드 |

### 유틸리티 (2종)

| 스킬명 | 역할 |
|:---|:---|
| `ai-dlc-program-spec` | 프로그램 명세서 생성 |
| `ai-dlc-md-to-word` | 마크다운 → Word 문서 변환 |

### 기타 (1종)

| 스킬명 | 역할 |
|:---|:---|
| `find-skills` | 작업에 맞는 스킬 탐색·안내 |

---

## 7. ID 체계 및 산출물 연계

각 산출물은 고유 ID를 가지며, 후속 스킬의 입력으로 연결됩니다. ID 체계를 일관되게 사용하면 `ai-dlc-consistency-check` 스킬이 연계 오류를 자동으로 탐지해 줍니다.

```
요구사항 정의서
  FR-NNN   기능 요구사항
  PR-NNN   성능 요구사항
  SR-NNN   보안 요구사항
  QR-NNN   품질 요구사항
  IR-NNN   인터페이스 요구사항
  DR-NNN   데이터 요구사항
  CR-NNN   제약 요구사항 (변경관리에서는 Change Request)
        │
        ▼
서비스 카탈로그 (SC-NNN) ← FR-NNN 참조
        │
        ▼
비즈니스 규칙 (BR-NNN) ← FR-NNN 참조
도메인 용어사전 (TM-NNN)
        │
        ▼
유즈케이스 (UC-NNN) ← FR-NNN, SC-NNN 참조
        │
   ┌────┴──────────────────────────────┐
   ▼                                   ▼
화면 (SCR-NNN)                   API (operationId)
← UC-NNN 참조                    ← UC-NNN, SCR-NNN 참조
        │                              │
        └────────────┬─────────────────┘
                     ▼
           데이터 설계 (테이블명) ← UC-NNN 참조
                     │
                     ▼
           클래스 설계 (CLS-NNN) ← UC-NNN, operationId, 테이블명 참조
                     │
                     ▼
           시퀀스 (SEQ-NNN) ← UC-NNN, CLS-NNN 참조
                     │
                     ▼
          [개발 단계 — 아키텍처별 분기]
```

### ID 연계 규칙

- 설계서에서 참조하는 ID는 반드시 이전 단계 산출물에서 확인합니다
- UC-NNN이 없으면 화면설계·API설계를 시작하지 않는 것을 권장합니다
- `operationId`는 API 설계서 기준이며, 코드 생성 시 메서드명·엔드포인트 기준이 됩니다
- `ai-dlc-consistency-check`로 ID 불일치를 주기적으로 검사하세요

---

## 8. 산출물 파일 규칙

### 파일명 형식

```
{산출물유형}_{사업명}_{YYYYMMDD}.md
```

**예시**

- `요구사항정의서_쇼핑몰프로젝트_20260524.md`
- `유즈케이스_쇼핑몰프로젝트_20260524.md`
- `openapi_쇼핑몰프로젝트_20260524.yaml`
- `Vue구현계획_쇼핑몰프로젝트_20260524.md`

수정(revise) 산출물은 버전 번호를 추가합니다.

- `비즈니스규칙_쇼핑몰프로젝트_20260524_v0.2.md`

### 저장 위치

기본적으로 **현재 작업 디렉터리**에 저장됩니다. 스킬 실행 후 절대 경로가 채팅창에 출력됩니다.

### 파일 생성 없이 채팅창에만 출력하기

```
"보여만 줘", "파일 만들지 마", "여기에 출력해줘", "대화창에 보여줘"
```

위 키워드를 포함하면 파일을 만들지 않고 채팅창에 내용을 출력합니다.

---

## 9. 스킬 사용 팁

### 파일 첨부로 컨텍스트 제공

`@파일명` 형식으로 기존 산출물을 스킬에 전달할 수 있습니다.

```
"@요구사항정의서_20260524.md 를 보고 유즈케이스 만들어줘"
"@openapi_20260524.yaml 기반으로 Spring Boot Controller 만들어줘"
"@화면설계서_20260524.md 를 참고해서 Vue 컴포넌트 만들어줘"
```

### 검증 → 수정 반복 패턴

모든 generate 스킬은 validate → revise 쌍을 가지고 있습니다. 이 패턴을 반복하여 품질을 높입니다.

```
1. 생성 (create/gen)    →  산출물 초안 작성
2. 검증 (validate)      →  이슈 목록 확인
3. 수정 (revise)        →  이슈 반영
4. 다시 2번으로          →  기준 충족까지 반복
```

### 코드 생성 후 품질 관리 패턴

```
코드 생성 (component-gen / page-gen / controller-gen 등)
    │
    ▼
code-review → 이슈 코드 목록 확인 (RV-NNN / SB-NNN / VV-NNN / NX-NNN)
ts-check    → TypeScript 오류 확인
lint-check  → ESLint 규칙 위반 확인
    │
    ▼
code-revise → 우선순위 순서로 수정
    │
    ▼
e2e-test-gen      → 통합 테스트 코드 생성
e2e-test-validate → 테스트 코드 품질 확인
e2e-test-revise   → 테스트 코드 보완
```

### 가이드 스킬 활용

가이드 스킬은 파일을 생성하지 않고 사용 방법·코드 예제를 채팅창에 바로 출력합니다.

```
"Pinia 스토어 사용법 알려줘"
"Auth.js v5 로그인 구현 방법 알려줘"
"VeeValidate Zod 폼 검증 방법 알려줘"
"Liquibase changelog 작성법 알려줘"
```

---

## 10. 자주 묻는 질문 (FAQ)

**Q. 어떤 스킬을 써야 할지 모를 때**
> "지금 어떤 스킬을 사용하면 되나요?" 라고 물어보면 현재 상황에 맞는 스킬을 안내해 줍니다. 또는 "설치된 AI-DLC 스킬 목록 보여줘"로 전체 목록을 확인하세요.

**Q. React와 Next.js 중 어떤 것을 써야 하나요?**
> SEO가 필요하거나, 서버 사이드 렌더링·인증·풀스택이 필요하면 **Next.js**. 단순 SPA이고 별도 API 서버가 있으면 **React/Vite**.

**Q. React와 Vue.js 중 어떤 것을 써야 하나요?**
> 팀의 기술 스택에 따라 선택하세요. 기능 면에서는 동등합니다. React 경험이 있으면 React/Vite 또는 Next.js, Vue.js 경험이 있거나 Options API → Composition API 마이그레이션 중이면 Vue.js 3.

**Q. Next.js에서 `'use client'`를 언제 붙이나요?**
> `useState`, `useEffect`, `onClick` 등 클라이언트 인터랙션이 필요할 때만 붙입니다. 데이터만 조회하는 컴포넌트는 RSC(기본값)로 둡니다.

**Q. Vue.js에서 `storeToRefs`를 왜 써야 하나요?**
> Pinia 스토어에서 state·getters를 구조분해하면 반응성이 깨집니다. `storeToRefs(store)`를 사용하면 ref로 감싸져 반응성이 유지됩니다.
> ```typescript
> // ❌ 반응성 깨짐
> const { user, isLoggedIn } = authStore
> // ✅ 반응성 유지
> const { user, isLoggedIn } = storeToRefs(authStore)
> ```

**Q. Vue.js에서 `v-for`에 index를 key로 쓰면 안 되나요?**
> 목록 순서가 바뀌거나 항목이 추가/삭제될 때 index key는 잘못된 컴포넌트 재사용을 일으킵니다. 반드시 `item.id`처럼 고유한 데이터 식별자를 사용하세요.

**Q. Server Actions vs Route Handlers 차이는 무엇인가요?**
> **Server Actions**: 폼 제출·내부 DB 뮤테이션. Next.js 앱 내부에서만 사용.  
> **Route Handlers**: 외부 클라이언트(모바일 앱·타 서비스) API 노출, 파일 업로드, 웹훅 수신.

**Q. 기존 코드에 스킬을 적용할 때**
> `@파일명`으로 기존 파일을 첨부하거나 "현재 프로젝트의 코드를 보고 ~해줘"라고 입력하세요.

**Q. 변경 요청이 발생했을 때 어디서부터 시작하나요?**
> `ai-dlc-change-register`로 CR을 먼저 등록하고, `ai-dlc-impact-analysis`로 영향 범위를 파악한 다음 수정을 시작하세요. 완료 후 `ai-dlc-consistency-check`로 ID 연계가 깨지지 않았는지 확인하세요.

---

## 11. 이 저장소 구조

```
ai-dlc/
├── README.md                   ← 이 파일 (GitHub 메인 가이드)
├── AI-DLC-README.md            ← 초기 작성 상세 가이드 (참조용)
├── CLAUDE.md                   ← Claude Code 작업 지침
├── plans/                      ← PLAN-001~009 계획 이력
│   ├── README.md               ← 계획 목록 인덱스
│   ├── PLAN-001_ai-dlc-requirements-skill.md
│   ├── PLAN-002_ai-dlc-analysis-skills-batch.md
│   ├── PLAN-003_code-analysis-skills-batch.md
│   ├── PLAN-004_design-skills-batch.md
│   ├── PLAN-005_sb-backend-skills-batch.md
│   ├── PLAN-006_fe-react-skills-batch.md
│   ├── PLAN-007_nxt-nextjs-skills-batch.md
│   └── PLAN-008_vue-skills-batch.md
└── skills/                     ← 110개 스킬 디렉터리
    ├── ai-dlc-common/          ← 공용 참조 문서 (artifacts-flow.md 등)
    │   └── references/
    ├── ai-dlc-requirements/
    ├── ai-dlc-service-catalog/
    ├── ai-dlc-biz-rules-*/     ← 분석 스킬
    ├── ai-dlc-glossary-*/
    ├── ai-dlc-usecase-*/       ← 설계 스킬
    ├── ai-dlc-screen-*/
    ├── ai-dlc-api-*/
    ├── ai-dlc-data-*/
    ├── ai-dlc-class-*/
    ├── ai-dlc-change-*/        ← 변경 관리 스킬
    ├── ai-dlc-sb-*/            ← Spring Boot 스킬
    ├── ai-dlc-fe-*/            ← React/Vite 스킬 (Next.js·Vue.js 공유)
    ├── ai-dlc-nxt-*/           ← Next.js 스킬
    ├── ai-dlc-vue-*/           ← Vue.js 3 스킬
    └── find-skills/
```

### 스킬 파일 구조

각 스킬 디렉터리는 다음 파일로 구성됩니다.

```
ai-dlc-{스킬명}/
├── SKILL.md        ← 스킬 정의 (트리거, 절차, 출력 형식)
└── template.md     ← 코드/문서 템플릿 (generate 스킬만 보유)
```

---

> 스킬 추가·변경 시 `plans/` 에 계획 파일을 작성하고 이 README.md도 함께 업데이트하세요.  
> 문의·기여는 [GitHub Issues](https://github.com/kdkim2000/ai-dlc/issues) 를 활용하세요.
