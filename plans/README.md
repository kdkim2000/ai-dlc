# AI-DLC Plans 이력

이 폴더는 AI-DLC 방법론 관련 Claude Code 작업에서 Plan 모드로 수립·승인·실행된 계획의 전체 이력을 보관한다.

## 파일 명명 규칙

```
PLAN-{NNN}_{주제_식별자}.md
```

- `NNN`: 3자리 순번 (001부터)
- `주제_식별자`: kebab-case 요약

## 계획 목록

| 번호 | 파일 | 제목 | 작성일 | 상태 |
|:---:|:---|:---|:---:|:---:|
| 001 | [PLAN-001_ai-dlc-requirements-skill.md](PLAN-001_ai-dlc-requirements-skill.md) | AI-DLC 요구사항 정의서 작성 스킬 생성 | 2026-05-23 | 완료 |
| 002 | [PLAN-002_ai-dlc-analysis-skills-batch.md](PLAN-002_ai-dlc-analysis-skills-batch.md) | AI-DLC 분석단계 스킬 10종 일괄 생성 | 2026-05-23 | 완료 |
| 003 | [PLAN-003_code-analysis-skills-batch.md](PLAN-003_code-analysis-skills-batch.md) | AI-DLC 코드분석단계 스킬 8종 일괄 생성 | 2026-05-23 | 완료 |
| 004 | [PLAN-004_design-skills-batch.md](PLAN-004_design-skills-batch.md) | AI-DLC 설계단계 스킬 18종 일괄 생성 | 2026-05-23 | 완료 |
| 005 | [PLAN-005_sb-backend-skills-batch.md](PLAN-005_sb-backend-skills-batch.md) | AI-DLC 개발단계(백엔드-Spring Boot) 스킬 23종 일괄 생성 | 2026-05-23 | 완료 |
| 006 | [PLAN-006_fe-react-skills-batch.md](PLAN-006_fe-react-skills-batch.md) | AI-DLC 개발단계(프론트엔드-React) 스킬 18종 일괄 생성 | 2026-05-23 | 완료 |
| 007 | [PLAN-007_nxt-nextjs-skills-batch.md](PLAN-007_nxt-nextjs-skills-batch.md) | AI-DLC 개발단계(프론트엔드-Next.js) 스킬 13종 일괄 생성 | 2026-05-23 | 완료 |
| 008 | [PLAN-008_vue-skills-batch.md](PLAN-008_vue-skills-batch.md) | AI-DLC 개발단계(프론트엔드-Vue.js) 스킬 14종 일괄 생성 | 2026-05-24 | 완료 |
| 009 | [PLAN-009_readme-creation.md](PLAN-009_readme-creation.md) | AI-DLC GitHub README.md 생성 | 2026-05-24 | 완료 |
| 010 | [PLAN-010_ai-dlc-readme-vue-update.md](PLAN-010_ai-dlc-readme-vue-update.md) | AI-DLC-README.md Vue.js 내용 최신화 | 2026-05-24 | 완료 |
| 011 | [PLAN-011_pre-requirements-skills-batch.md](PLAN-011_pre-requirements-skills-batch.md) | AI-DLC Pre-Requirements 스킬 5종 일괄 생성 | 2026-05-25 | 완료 |

## 산출물 위치

모든 스킬 파일은 `C:\Users\kdkim2000\.claude\skills\` 하위에 생성됨.

| 스킬명 | 디렉터리 |
|:---|:---|
| ai-dlc-requirements | `.claude\skills\ai-dlc-requirements\` |
| ai-dlc-common (공용) | `.claude\skills\ai-dlc-common\` |
| ai-dlc-service-catalog | `.claude\skills\ai-dlc-service-catalog\` |
| ai-dlc-functional-req | `.claude\skills\ai-dlc-functional-req\` |
| ai-dlc-nonfunctional-req | `.claude\skills\ai-dlc-nonfunctional-req\` |
| ai-dlc-biz-rules-create | `.claude\skills\ai-dlc-biz-rules-create\` |
| ai-dlc-biz-rules-validate | `.claude\skills\ai-dlc-biz-rules-validate\` |
| ai-dlc-biz-rules-revise | `.claude\skills\ai-dlc-biz-rules-revise\` |
| ai-dlc-glossary-create | `.claude\skills\ai-dlc-glossary-create\` |
| ai-dlc-glossary-validate | `.claude\skills\ai-dlc-glossary-validate\` |
| ai-dlc-glossary-revise | `.claude\skills\ai-dlc-glossary-revise\` |
| ai-dlc-glossary-apply | `.claude\skills\ai-dlc-glossary-apply\` |
| ai-dlc-impact-analysis | `.claude\skills\ai-dlc-impact-analysis\` |
| ai-dlc-md-to-word | `.claude\skills\ai-dlc-md-to-word\` |
| ai-dlc-program-spec | `.claude\skills\ai-dlc-program-spec\` |
| ai-dlc-code-traceability | `.claude\skills\ai-dlc-code-traceability\` |
| ai-dlc-dependency-analysis | `.claude\skills\ai-dlc-dependency-analysis\` |
| ai-dlc-code-complexity | `.claude\skills\ai-dlc-code-complexity\` |
| ai-dlc-api-spec-extract | `.claude\skills\ai-dlc-api-spec-extract\` |
| ai-dlc-data-model-analysis | `.claude\skills\ai-dlc-data-model-analysis\` |
| ai-dlc-design-extract | `.claude\skills\ai-dlc-design-extract\` |
| ai-dlc-usecase-create | `.claude\skills\ai-dlc-usecase-create\` |
| ai-dlc-usecase-validate | `.claude\skills\ai-dlc-usecase-validate\` |
| ai-dlc-usecase-revise | `.claude\skills\ai-dlc-usecase-revise\` |
| ai-dlc-screen-list | `.claude\skills\ai-dlc-screen-list\` |
| ai-dlc-screen-spec | `.claude\skills\ai-dlc-screen-spec\` |
| ai-dlc-screen-validate | `.claude\skills\ai-dlc-screen-validate\` |
| ai-dlc-screen-revise | `.claude\skills\ai-dlc-screen-revise\` |
| ai-dlc-api-design | `.claude\skills\ai-dlc-api-design\` |
| ai-dlc-api-validate | `.claude\skills\ai-dlc-api-validate\` |
| ai-dlc-api-revise | `.claude\skills\ai-dlc-api-revise\` |
| ai-dlc-data-design | `.claude\skills\ai-dlc-data-design\` |
| ai-dlc-data-validate | `.claude\skills\ai-dlc-data-validate\` |
| ai-dlc-data-revise | `.claude\skills\ai-dlc-data-revise\` |
| ai-dlc-class-design | `.claude\skills\ai-dlc-class-design\` |
| ai-dlc-class-validate | `.claude\skills\ai-dlc-class-validate\` |
| ai-dlc-class-revise | `.claude\skills\ai-dlc-class-revise\` |
| ai-dlc-sequence-design | `.claude\skills\ai-dlc-sequence-design\` |
| ai-dlc-sb-project-setup | `.claude\skills\ai-dlc-sb-project-setup\` |
| ai-dlc-sb-anyframe-setup | `.claude\skills\ai-dlc-sb-anyframe-setup\` |
| ai-dlc-sb-migration-plan | `.claude\skills\ai-dlc-sb-migration-plan\` |
| ai-dlc-sb-sql-plan | `.claude\skills\ai-dlc-sb-sql-plan\` |
| ai-dlc-sb-sql-gen | `.claude\skills\ai-dlc-sb-sql-gen\` |
| ai-dlc-sb-migration-exec | `.claude\skills\ai-dlc-sb-migration-exec\` |
| ai-dlc-sb-layer-impl | `.claude\skills\ai-dlc-sb-layer-impl\` |
| ai-dlc-sb-vo-gen | `.claude\skills\ai-dlc-sb-vo-gen\` |
| ai-dlc-sb-mapper-gen | `.claude\skills\ai-dlc-sb-mapper-gen\` |
| ai-dlc-sb-service-gen | `.claude\skills\ai-dlc-sb-service-gen\` |
| ai-dlc-sb-controller-gen | `.claude\skills\ai-dlc-sb-controller-gen\` |
| ai-dlc-sb-unit-test-gen | `.claude\skills\ai-dlc-sb-unit-test-gen\` |
| ai-dlc-sb-unit-test-validate | `.claude\skills\ai-dlc-sb-unit-test-validate\` |
| ai-dlc-sb-unit-test-revise | `.claude\skills\ai-dlc-sb-unit-test-revise\` |
| ai-dlc-sb-code-review | `.claude\skills\ai-dlc-sb-code-review\` |
| ai-dlc-sb-code-revise | `.claude\skills\ai-dlc-sb-code-revise\` |
| ai-dlc-sb-springboot-guide | `.claude\skills\ai-dlc-sb-springboot-guide\` |
| ai-dlc-sb-anyframe-guide | `.claude\skills\ai-dlc-sb-anyframe-guide\` |
| ai-dlc-sb-mybatis-guide | `.claude\skills\ai-dlc-sb-mybatis-guide\` |
| ai-dlc-sb-dbio-guide | `.claude\skills\ai-dlc-sb-dbio-guide\` |
| ai-dlc-sb-security-guide | `.claude\skills\ai-dlc-sb-security-guide\` |
| ai-dlc-sb-db-guide | `.claude\skills\ai-dlc-sb-db-guide\` |
| ai-dlc-sb-liquibase-guide | `.claude\skills\ai-dlc-sb-liquibase-guide\` |
| ai-dlc-fe-project-setup | `.claude\skills\ai-dlc-fe-project-setup\` |
| ai-dlc-fe-node-setup | `.claude\skills\ai-dlc-fe-node-setup\` |
| ai-dlc-fe-impl-plan | `.claude\skills\ai-dlc-fe-impl-plan\` |
| ai-dlc-fe-component-gen | `.claude\skills\ai-dlc-fe-component-gen\` |
| ai-dlc-fe-e2e-test-gen | `.claude\skills\ai-dlc-fe-e2e-test-gen\` |
| ai-dlc-fe-code-review | `.claude\skills\ai-dlc-fe-code-review\` |
| ai-dlc-fe-ts-check | `.claude\skills\ai-dlc-fe-ts-check\` |
| ai-dlc-fe-lint-check | `.claude\skills\ai-dlc-fe-lint-check\` |
| ai-dlc-fe-e2e-test-validate | `.claude\skills\ai-dlc-fe-e2e-test-validate\` |
| ai-dlc-fe-code-revise | `.claude\skills\ai-dlc-fe-code-revise\` |
| ai-dlc-fe-e2e-test-revise | `.claude\skills\ai-dlc-fe-e2e-test-revise\` |
| ai-dlc-fe-shadcn-guide | `.claude\skills\ai-dlc-fe-shadcn-guide\` |
| ai-dlc-fe-tailwind-guide | `.claude\skills\ai-dlc-fe-tailwind-guide\` |
| ai-dlc-fe-axios-guide | `.claude\skills\ai-dlc-fe-axios-guide\` |
| ai-dlc-fe-zod-guide | `.claude\skills\ai-dlc-fe-zod-guide\` |
| ai-dlc-fe-perf-guide | `.claude\skills\ai-dlc-fe-perf-guide\` |
| ai-dlc-fe-state-guide | `.claude\skills\ai-dlc-fe-state-guide\` |
| ai-dlc-fe-react-query-guide | `.claude\skills\ai-dlc-fe-react-query-guide\` |
| ai-dlc-nxt-project-setup | `.claude\skills\ai-dlc-nxt-project-setup\` |
| ai-dlc-nxt-impl-plan | `.claude\skills\ai-dlc-nxt-impl-plan\` |
| ai-dlc-nxt-page-gen | `.claude\skills\ai-dlc-nxt-page-gen\` |
| ai-dlc-nxt-route-handler-gen | `.claude\skills\ai-dlc-nxt-route-handler-gen\` |
| ai-dlc-nxt-server-action-gen | `.claude\skills\ai-dlc-nxt-server-action-gen\` |
| ai-dlc-nxt-e2e-test-gen | `.claude\skills\ai-dlc-nxt-e2e-test-gen\` |
| ai-dlc-nxt-code-review | `.claude\skills\ai-dlc-nxt-code-review\` |
| ai-dlc-nxt-code-revise | `.claude\skills\ai-dlc-nxt-code-revise\` |
| ai-dlc-nxt-sc-guide | `.claude\skills\ai-dlc-nxt-sc-guide\` |
| ai-dlc-nxt-auth-guide | `.claude\skills\ai-dlc-nxt-auth-guide\` |
| ai-dlc-nxt-middleware-guide | `.claude\skills\ai-dlc-nxt-middleware-guide\` |
| ai-dlc-nxt-perf-guide | `.claude\skills\ai-dlc-nxt-perf-guide\` |
| ai-dlc-nxt-deploy-guide | `.claude\skills\ai-dlc-nxt-deploy-guide\` |
| ai-dlc-vue-project-setup | `.claude\skills\ai-dlc-vue-project-setup\` |
| ai-dlc-vue-impl-plan | `.claude\skills\ai-dlc-vue-impl-plan\` |
| ai-dlc-vue-component-gen | `.claude\skills\ai-dlc-vue-component-gen\` |
| ai-dlc-vue-e2e-test-gen | `.claude\skills\ai-dlc-vue-e2e-test-gen\` |
| ai-dlc-vue-code-review | `.claude\skills\ai-dlc-vue-code-review\` |
| ai-dlc-vue-ts-check | `.claude\skills\ai-dlc-vue-ts-check\` |
| ai-dlc-vue-lint-check | `.claude\skills\ai-dlc-vue-lint-check\` |
| ai-dlc-vue-code-revise | `.claude\skills\ai-dlc-vue-code-revise\` |
| ai-dlc-vue-pinia-guide | `.claude\skills\ai-dlc-vue-pinia-guide\` |
| ai-dlc-vue-router-guide | `.claude\skills\ai-dlc-vue-router-guide\` |
| ai-dlc-vue-query-guide | `.claude\skills\ai-dlc-vue-query-guide\` |
| ai-dlc-vue-form-guide | `.claude\skills\ai-dlc-vue-form-guide\` |
| ai-dlc-vue-ui-guide | `.claude\skills\ai-dlc-vue-ui-guide\` |
| ai-dlc-vue-perf-guide | `.claude\skills\ai-dlc-vue-perf-guide\` |
| ai-dlc-idea-clarify | `.claude\skills\ai-dlc-idea-clarify\` |
| ai-dlc-persona-create | `.claude\skills\ai-dlc-persona-create\` |
| ai-dlc-user-story-map | `.claude\skills\ai-dlc-user-story-map\` |
| ai-dlc-mvp-scope | `.claude\skills\ai-dlc-mvp-scope\` |
| ai-dlc-idea-to-req | `.claude\skills\ai-dlc-idea-to-req\` |
