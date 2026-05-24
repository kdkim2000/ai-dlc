# AI-DLC 분석단계 산출물 흐름도

## 단계별 산출물 흐름

```
[요구사항 정의서] ← ai-dlc-requirements 스킬이 생성
       │
       ▼
[서비스 카탈로그 분류] ← ai-dlc-service-catalog
       │
       ├──────────────────────┐
       ▼                      ▼
[기능 요구사항 정의]    [비기능 요구사항 정의]
ai-dlc-functional-req   ai-dlc-nonfunctional-req
       │                      │
       └──────────┬───────────┘
                  ▼
     ┌────────────┴────────────┐
     ▼                         ▼
[비즈니스 규칙 생성]    [도메인 표준 용어 생성]
ai-dlc-biz-rules-create  ai-dlc-glossary-create
     │                         │
     ▼                         ▼
[비즈니스 규칙 검증]    [도메인 표준 용어 검증]
ai-dlc-biz-rules-validate ai-dlc-glossary-validate
     │                         │
     ▼                         ▼
[비즈니스 규칙 수정]    [도메인 표준 용어 수정]
ai-dlc-biz-rules-revise  ai-dlc-glossary-revise
                               │
                               ▼
              [서비스 요구사항 표준 용어 반영]
              ai-dlc-glossary-apply
                               │
                               ▼
              [표준화된 요구사항 정의서 v2]
```

## 산출물 간 ID 연계

| 산출물 | ID 체계 | 연계 |
|:---|:---|:---|
| 요구사항 정의서 | FR-NNN, PR-NNN, SR-NNN, QR-NNN, IR-NNN, DR-NNN, CR-NNN | 기준 문서 |
| 서비스 카탈로그 | SC-NNN (서비스) | FR-NNN 요구사항 참조 |
| 기능 요구사항 | FR-NNN | 요구사항 정의서와 동일 체계 |
| 비기능 요구사항 | PR/SR/QR/IR/DR/CR-NNN | 요구사항 정의서와 동일 체계 |
| 비즈니스 규칙 | BR-NNN | FR-NNN, SC-NNN 참조 |
| 도메인 표준 용어 | TM-NNN (용어) | 모든 산출물에서 참조 |

## 스킬 활용 순서 (권장)

1. `ai-dlc-requirements` → 요구사항 정의서 생성
2. `ai-dlc-service-catalog` → 서비스 단위 분류
3. `ai-dlc-functional-req` 또는 `ai-dlc-nonfunctional-req` → 요구사항 심화 정의
4. `ai-dlc-biz-rules-create` → 비즈니스 규칙 도출
5. `ai-dlc-biz-rules-validate` → 규칙 검증
6. `ai-dlc-biz-rules-revise` → 규칙 수정 (필요 시 5↔6 반복)
7. `ai-dlc-glossary-create` → 도메인 용어사전 생성
8. `ai-dlc-glossary-validate` → 용어사전 검증
9. `ai-dlc-glossary-revise` → 용어사전 수정 (필요 시 8↔9 반복)
10. `ai-dlc-glossary-apply` → 요구사항 문서에 표준 용어 반영

---

## 설계단계 산출물 흐름 (PLAN-004)

```
[분석단계 산출물]
  요구사항(FR) + 서비스 카탈로그(SC) + 도메인 용어사전
  + 코드분석 산출물 (ai-dlc-api-spec-extract, ai-dlc-data-model-analysis 등)
        │
        ▼ [선택] 레거시 시스템인 경우
  ai-dlc-design-extract → 설계기초추출서
        │
        ▼
  ai-dlc-usecase-create → UC-NNN 유즈케이스 문서
  → ai-dlc-usecase-validate / ai-dlc-usecase-revise 반복
        │
   ┌────┴─────────────────────────┐
   ▼                              ▼
ai-dlc-screen-list(SCR)    ai-dlc-api-design(YAML+MD)
→ ai-dlc-screen-spec       → ai-dlc-api-validate
→ ai-dlc-screen-validate   → ai-dlc-api-revise 반복
→ ai-dlc-screen-revise 반복
        │                         │
        └──────────┬──────────────┘
                   ▼
           ai-dlc-data-design → 데이터설계서
           → ai-dlc-data-validate / ai-dlc-data-revise 반복
                   │
                   ▼
           ai-dlc-class-design → 클래스설계서
           → ai-dlc-class-validate / ai-dlc-class-revise 반복
                   │
                   ▼
           ai-dlc-sequence-design → 시퀀스다이어그램
```

## 설계단계 산출물 간 ID 연계

| 산출물 | ID 체계 | 연계 |
|:---|:---|:---|
| 유즈케이스 | UC-NNN | FR-NNN, SC-NNN 참조 |
| 화면 | SCR-NNN | UC-NNN 참조 |
| API | operationId | UC-NNN, SCR-NNN 참조 |
| 데이터 | 테이블명 | UC-NNN 참조 |
| 클래스 | CLS-NNN | UC-NNN, operationId, 테이블명 참조 |
| 시퀀스 다이어그램 | SEQ-NNN | UC-NNN, CLS-NNN 참조 |
| 검증 이슈 | VI-NNN | validate 스킬 공통 |

---

## 개발단계(백엔드-Spring Boot) 산출물 흐름 (PLAN-005)

```
[설계단계 산출물 (PLAN-004)]
  클래스설계서(CLS-NNN) + 데이터설계서(테이블 정의) + API설계서(operationId)
        │
   ┌────┴──────────────────────────────┐
   ▼                                   ▼
ai-dlc-sb-project-setup            ai-dlc-sb-migration-plan
ai-dlc-sb-anyframe-setup           → ai-dlc-sb-sql-plan
(pom.xml, application-*.yml,       → ai-dlc-sb-sql-gen
 Anyframe 설정 일체)                  (DDL/DML .sql 파일)
                                   → ai-dlc-sb-migration-exec
                                      (changelog.xml / V*.sql)
        │                                      │
        └─────────────────┬────────────────────┘
                          ▼
             ai-dlc-sb-layer-impl (오케스트레이터)
                  순서대로 스킬 실행 안내
                          │
              ┌───────────┼───────────────────┐
              ▼           ▼                   ▼
        ai-dlc-sb-vo-gen  ai-dlc-sb-mapper-gen ai-dlc-sb-service-gen
        (*VO.java,         (*Mapper.java +       (*Service.java +
         *ReqVO.java,       MyBatis XML)          *ServiceImpl.java)
         *SearchVO.java)                                │
                                                        ▼
                                               ai-dlc-sb-controller-gen
                                               (*Controller.java,
                                                ApiResponse<T>)
                          │
                          ▼
             ai-dlc-sb-unit-test-gen (*Test.java)
             → ai-dlc-sb-unit-test-validate (검증 보고서)
             → ai-dlc-sb-unit-test-revise  (수정된 테스트)
                          │
                          ▼
             ai-dlc-sb-code-review (코드품질검토_*.md)
             → ai-dlc-sb-code-revise (수정된 소스코드)

[가이드 스킬] — 개발 진행 중 수시로 참조 (대화창 직접 출력)
  ai-dlc-sb-springboot-guide  ai-dlc-sb-anyframe-guide
  ai-dlc-sb-mybatis-guide     ai-dlc-sb-dbio-guide
  ai-dlc-sb-security-guide    ai-dlc-sb-db-guide
  ai-dlc-sb-liquibase-guide
```

## 개발단계 산출물 간 연계

| 산출물 | 스킬 | 입력 설계서 |
|:---|:---|:---|
| pom.xml, application-*.yml | ai-dlc-sb-project-setup | 기술스택 정의 |
| Anyframe 설정 파일 | ai-dlc-sb-anyframe-setup | project-setup 산출물 |
| DB마이그레이션계획_*.md | ai-dlc-sb-migration-plan | 데이터설계서 |
| SQL생성계획_*.md | ai-dlc-sb-sql-plan | 데이터설계서 |
| DDL/DML .sql 파일 | ai-dlc-sb-sql-gen | sql-plan 산출물 |
| changelog.xml / V*.sql | ai-dlc-sb-migration-exec | sql-gen 산출물 |
| *VO.java, *ReqVO.java | ai-dlc-sb-vo-gen | 데이터설계서, CLS-NNN |
| *Mapper.java + XML | ai-dlc-sb-mapper-gen | VO, 데이터설계서 |
| *Service.java + Impl | ai-dlc-sb-service-gen | CLS-NNN, operationId, Mapper |
| *Controller.java | ai-dlc-sb-controller-gen | operationId, API설계서, Service |
| *Test.java | ai-dlc-sb-unit-test-gen | Service/Controller/Mapper 코드 |
| 테스트코드_검증_*.md | ai-dlc-sb-unit-test-validate | 테스트 코드 |
| 코드품질검토_*.md | ai-dlc-sb-code-review | 생성된 소스코드 전체 |

## 스킬 활용 순서 (개발단계-백엔드 권장)

1. `ai-dlc-sb-project-setup` → 프로젝트 초기 설정
2. `ai-dlc-sb-anyframe-setup` → Anyframe 통합 (선택)
3. `ai-dlc-sb-migration-plan` → DB 마이그레이션 전략
4. `ai-dlc-sb-sql-plan` → SQL 생성 계획
5. `ai-dlc-sb-sql-gen` → DDL/DML SQL 파일 생성
6. `ai-dlc-sb-migration-exec` → Liquibase/Flyway 설정 실행
7. `ai-dlc-sb-layer-impl` → 레이어 구현 순서 안내
8. `ai-dlc-sb-vo-gen` → VO 코드 생성
9. `ai-dlc-sb-mapper-gen` → Mapper 코드 생성
10. `ai-dlc-sb-service-gen` → Service 코드 생성
11. `ai-dlc-sb-controller-gen` → Controller 코드 생성
12. `ai-dlc-sb-unit-test-gen` → 단위 테스트 생성
13. `ai-dlc-sb-unit-test-validate` → 테스트 코드 검증
14. `ai-dlc-sb-unit-test-revise` → 테스트 코드 수정 (필요 시 13↔14 반복)
15. `ai-dlc-sb-code-review` → 전체 코드 품질 검토
16. `ai-dlc-sb-code-revise` → 코드 수정 반영 (필요 시 15↔16 반복)

---

## 개발단계(프론트엔드-React) 산출물 흐름 (PLAN-006)

```
[설계단계 산출물 (PLAN-004)]
  화면설계서(SCR-NNN) + API설계서(operationId) + 유즈케이스(UC-NNN)
        │
   ┌────┴──────────────────────────────┐
   ▼                                   ▼
ai-dlc-fe-project-setup            ai-dlc-fe-node-setup
(package.json, vite.config.ts,     (Express BFF/Mock 서버)
 tsconfig.json strict, ESLint,      src/app.ts, routes/,
 Tailwind, shadcn/ui, src/lib/)     ApiResponse<T>)
        │                                  │
        └──────────────┬───────────────────┘
                       ▼
            ai-dlc-fe-impl-plan
            (프론트엔드구현계획_*.md)
            화면 목록, 구현 단계, queryKey 설계, API 함수 패턴
                       │
                       ▼
            ai-dlc-fe-component-gen
            (pages/*.tsx, components/*.tsx,
             hooks/useXxx.ts, api/xxx.api.ts,
             types/*.types.ts, types/*.schema.ts)
                       │
              ┌────────┴────────────────┐
              ▼                         ▼
   ai-dlc-fe-code-review         ai-dlc-fe-e2e-test-gen
   ai-dlc-fe-ts-check            (tests/pages/*.ts,
   ai-dlc-fe-lint-check           tests/*.spec.ts)
   (코드품질검토_*.md,                   │
    TypeScript검사결과_*.md,    ai-dlc-fe-e2e-test-validate
    ESLint검사결과_*.md)         (e2e테스트_검증_*.md)
              │                         │
              ▼                         ▼
   ai-dlc-fe-code-revise     ai-dlc-fe-e2e-test-revise
   (수정된 소스코드)          (수정된 테스트 코드)

[가이드 스킬] — 개발 진행 중 수시로 참조 (대화창 직접 출력)
  ai-dlc-fe-shadcn-guide    ai-dlc-fe-tailwind-guide
  ai-dlc-fe-axios-guide     ai-dlc-fe-zod-guide
  ai-dlc-fe-perf-guide      ai-dlc-fe-state-guide
  ai-dlc-fe-react-query-guide
```

## 개발단계(FE) 산출물 간 연계

| 산출물 | 스킬 | 입력 |
|:---|:---|:---|
| package.json, vite.config.ts, tsconfig.json | ai-dlc-fe-project-setup | 기술스택 정의 |
| src/app.ts, routes/ (Mock API) | ai-dlc-fe-node-setup | API 설계서 |
| 프론트엔드구현계획_*.md | ai-dlc-fe-impl-plan | SCR-NNN, operationId, project-setup 산출물 |
| pages/*.tsx, hooks/*.ts, api/*.ts | ai-dlc-fe-component-gen | SCR-NNN, operationId, impl-plan 산출물 |
| tests/**/*.spec.ts, tests/pages/*.ts | ai-dlc-fe-e2e-test-gen | UC-NNN, SCR-NNN, 구현 컴포넌트 |
| 코드품질검토_*.md | ai-dlc-fe-code-review | src/ 전체 소스코드 |
| TypeScript검사결과_*.md | ai-dlc-fe-ts-check | src/ 전체 소스코드 |
| ESLint검사결과_*.md | ai-dlc-fe-lint-check | src/ 전체 소스코드 |
| e2e테스트_검증_*.md | ai-dlc-fe-e2e-test-validate | tests/ 전체, UC 목록 |

## 스킬 활용 순서 (개발단계-프론트엔드 권장)

1. `ai-dlc-fe-project-setup` → React/Vite 프로젝트 초기 설정
2. `ai-dlc-fe-node-setup` → Node.js/Express Mock API 또는 BFF (선택)
3. `ai-dlc-fe-impl-plan` → 프론트엔드 구현 전략 계획
4. `ai-dlc-fe-component-gen` → 화면별 컴포넌트 코드 생성 (반복)
5. `ai-dlc-fe-code-review` → 생성된 코드 품질 검토
6. `ai-dlc-fe-ts-check` → TypeScript 타입 안전성 검사
7. `ai-dlc-fe-lint-check` → ESLint 코드 스타일 검사
8. `ai-dlc-fe-code-revise` → 리뷰 결과 반영 (필요 시 5↔8 반복)
9. `ai-dlc-fe-e2e-test-gen` → Playwright e2e 테스트 코드 생성
10. `ai-dlc-fe-e2e-test-validate` → e2e 테스트 코드 검증
11. `ai-dlc-fe-e2e-test-revise` → 테스트 수정 (필요 시 10↔11 반복)

---

## 개발단계(프론트엔드-Next.js) 산출물 흐름 (PLAN-007)

```
[설계단계 산출물 (PLAN-004)]
  화면설계서(SCR-NNN) + API설계서(operationId) + 유즈케이스(UC-NNN)
        │
        ▼
ai-dlc-nxt-project-setup
(package.json, next.config.ts, tsconfig.json,
 src/lib/auth.ts, src/middleware.ts, .env.example)
        │
        ▼
ai-dlc-nxt-impl-plan
(Next.js구현계획_*.md — 라우트 트리, RSC/CC 분류, 데이터 패칭 전략)
        │
   ┌────┴──────────────────────────────────┐
   ▼                                       ▼
ai-dlc-nxt-page-gen                ai-dlc-nxt-route-handler-gen
(app/**/page.tsx, layout.tsx,       (app/api/**/route.ts,
 loading.tsx, error.tsx,             src/types/api.ts)
 components/*.tsx)                         │
        │                                  ▼
        │                     ai-dlc-nxt-server-action-gen
        │                     (actions/*.ts,
        │                      components/*Form.tsx)
        └───────────┬─────────────────────┘
                    ▼
       ai-dlc-nxt-code-review  (코드품질검토_*.md)
       ai-dlc-fe-ts-check      (TypeScript검사결과_*.md)
       ai-dlc-fe-lint-check    (ESLint검사결과_*.md)
                    │
                    ▼
       ai-dlc-nxt-code-revise  (수정된 소스코드)
                    │
                    ▼
       ai-dlc-nxt-e2e-test-gen (tests/**/*.spec.ts)
       → ai-dlc-fe-e2e-test-validate
       → ai-dlc-fe-e2e-test-revise

[가이드 스킬] — 수시 참조 (대화창 직접 출력)
  ai-dlc-nxt-sc-guide          ai-dlc-nxt-auth-guide
  ai-dlc-nxt-middleware-guide  ai-dlc-nxt-perf-guide
  ai-dlc-nxt-deploy-guide
  [fe-* 재사용]: ai-dlc-fe-shadcn-guide, ai-dlc-fe-tailwind-guide,
                 ai-dlc-fe-zod-guide, ai-dlc-fe-state-guide,
                 ai-dlc-fe-react-query-guide
```

## 스킬 활용 순서 (개발단계-프론트엔드-Next.js 권장)

1. `ai-dlc-nxt-project-setup` → Next.js 15 App Router 프로젝트 초기 설정
2. `ai-dlc-nxt-impl-plan` → RSC/CC 분류 + 라우트 구조 + 데이터 패칭 전략 계획
3. `ai-dlc-nxt-page-gen` → 화면별 page.tsx, layout.tsx, 컴포넌트 코드 생성 (반복)
4. `ai-dlc-nxt-route-handler-gen` → app/api/ Route Handler 코드 생성 (필요 시)
5. `ai-dlc-nxt-server-action-gen` → actions/ Server Action 코드 생성 (반복)
6. `ai-dlc-nxt-code-review` → 생성된 코드 NX-001~010 + TC/SC/PF/A11Y 검토
7. `ai-dlc-fe-ts-check` → TypeScript 타입 안전성 검사
8. `ai-dlc-fe-lint-check` → ESLint 코드 스타일 검사
9. `ai-dlc-nxt-code-revise` → 리뷰 결과 반영 (필요 시 6↔9 반복)
10. `ai-dlc-nxt-e2e-test-gen` → Playwright e2e 테스트 코드 생성
11. `ai-dlc-fe-e2e-test-validate` → e2e 테스트 코드 검증
12. `ai-dlc-fe-e2e-test-revise` → 테스트 수정 (필요 시 11↔12 반복)

---

## 개발단계(프론트엔드-Vue.js) 산출물 흐름 (PLAN-008)

```
[설계단계 산출물 (PLAN-004)]
  화면설계서(SCR-NNN) + API설계서(operationId) + 유즈케이스(UC-NNN)
        │
        ▼
ai-dlc-vue-project-setup
(package.json, vite.config.ts, tsconfig.json,
 src/router/index.ts, src/stores/auth.ts, .env.example)
        │
        ▼
ai-dlc-vue-impl-plan
(Vue구현계획_*.md — 라우트 표, 컴포넌트 분류, Pinia 스토어 목록)
        │
        ▼
ai-dlc-vue-component-gen
(src/views/*.vue, src/components/*.vue,
 src/composables/useXxx.ts, src/stores/*.ts,
 src/api/*.api.ts, src/types/*.types.ts)
        │
        ▼
ai-dlc-vue-code-review  (코드품질검토_*.md)
ai-dlc-vue-ts-check     (TypeScript검사결과_*.md)
ai-dlc-vue-lint-check   (ESLint검사결과_*.md)
        │
        ▼
ai-dlc-vue-code-revise  (수정된 소스코드)
        │
        ▼
ai-dlc-vue-e2e-test-gen (tests/**/*.spec.ts)
→ ai-dlc-fe-e2e-test-validate
→ ai-dlc-fe-e2e-test-revise

[가이드 스킬] — 수시 참조 (대화창 직접 출력)
  ai-dlc-vue-pinia-guide    ai-dlc-vue-router-guide
  ai-dlc-vue-query-guide    ai-dlc-vue-form-guide
  ai-dlc-vue-ui-guide       ai-dlc-vue-perf-guide
  [fe-* 재사용]: ai-dlc-fe-tailwind-guide, ai-dlc-fe-axios-guide,
                 ai-dlc-fe-zod-guide, ai-dlc-fe-node-setup,
                 ai-dlc-fe-e2e-test-validate, ai-dlc-fe-e2e-test-revise
```

## 스킬 활용 순서 (개발단계-프론트엔드-Vue.js 권장)

1. `ai-dlc-vue-project-setup` → Vue 3 + Vite 프로젝트 초기 설정
2. `ai-dlc-fe-node-setup` → Node.js/Express Mock API 또는 BFF (선택)
3. `ai-dlc-vue-impl-plan` → 라우트 구조 + 컴포넌트 분류 + Pinia 스토어 계획
4. `ai-dlc-vue-component-gen` → View/컴포넌트/Composable 코드 생성 (반복)
5. `ai-dlc-vue-code-review` → 생성된 코드 VV-001~010 + TC/SC/PF/A11Y 검토
6. `ai-dlc-vue-ts-check` → vue-tsc TypeScript 타입 안전성 검사
7. `ai-dlc-vue-lint-check` → eslint-plugin-vue 코드 스타일 검사
8. `ai-dlc-vue-code-revise` → 리뷰 결과 반영 (필요 시 5↔8 반복)
9. `ai-dlc-vue-e2e-test-gen` → Playwright e2e 테스트 코드 생성
10. `ai-dlc-fe-e2e-test-validate` → e2e 테스트 코드 검증
11. `ai-dlc-fe-e2e-test-revise` → 테스트 수정 (필요 시 10↔11 반복)
