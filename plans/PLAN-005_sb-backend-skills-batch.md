# PLAN-005: AI-DLC 개발단계(백엔드-Spring Boot) 스킬 23종 일괄 생성

## 메타

| 항목 | 내용 |
|:---|:---|
| 작성일 | 2026-05-23 |
| 상태 | 완료 |
| 전제 플랜 | PLAN-004 (설계단계 스킬 18종) |
| 스킬 경로 | `C:\Users\kdkim2000\.claude\skills\ai-dlc-sb-*` |

---

## Context

PLAN-001~004 완료로 요구사항·분석·코드분석·설계 단계 스킬(총 38종)이 갖춰졌다. 이제 **개발단계(Backend-Spring Boot)**에 필요한 스킬을 일괄 생성한다. 설계단계 산출물(클래스설계서·데이터설계서·API설계서)을 Spring Boot 실제 소스코드·SQL·설정 파일로 변환하는 단계다.

---

## 사용자 요구사항

AI-DLC 개발(백엔드-Springboot)단계에서 필요한 skills 목록을 도출하고 일괄 생성:
- Spring Boot 프로젝트 설정, Anyframe 설정
- Liquibase/Flyway DB 마이그레이션 계획 및 실행
- ANSI SQL 생성 계획 및 DDL/DML 생성
- VO/Mapper/Service/Controller 레이어 코드 생성
- 단위 테스트 생성
- 코드 품질 검토 및 수정
- 개발 가이드라인 7종 (SpringBoot/Anyframe/MyBatis/DBIO/보안/DB/Liquibase)

---

## 설계 결정 사항

| 항목 | 결정 | 근거 |
|:---|:---|:---|
| 총 스킬 수 | 23종 | 사용자 지정 20종 + 제안 3종 (unit-test-validate, unit-test-revise 추가) |
| allowed-tools 분류 | 2-tier: create/revise=RGGwWE, validate/guide=RGG | 생성 스킬은 파일 쓰기 필요, 가이드/검토는 읽기만 |
| Anyframe/BXCM 가이드 | SKILL.md 내장 | 외부 참조 문서 미준비 |
| 마이그레이션 기본 도구 | Liquibase | Flyway 선택 가능 |
| 가이드 스킬 출력 | 대화창 직접 출력 | 파일 저장 없음 |

---

## 생성된 파일 구조

```
C:\Users\kdkim2000\.claude\skills\
├── ai-dlc-sb-project-setup\     SKILL.md + template.md
├── ai-dlc-sb-anyframe-setup\    SKILL.md + template.md
├── ai-dlc-sb-migration-plan\    SKILL.md + template.md
├── ai-dlc-sb-sql-plan\          SKILL.md + template.md
├── ai-dlc-sb-sql-gen\           SKILL.md + template.md
├── ai-dlc-sb-migration-exec\    SKILL.md + template.md
├── ai-dlc-sb-layer-impl\        SKILL.md
├── ai-dlc-sb-vo-gen\            SKILL.md + template.md
├── ai-dlc-sb-mapper-gen\        SKILL.md + template.md
├── ai-dlc-sb-service-gen\       SKILL.md + template.md
├── ai-dlc-sb-controller-gen\    SKILL.md + template.md
├── ai-dlc-sb-unit-test-gen\     SKILL.md + template.md
├── ai-dlc-sb-unit-test-validate\ SKILL.md
├── ai-dlc-sb-unit-test-revise\  SKILL.md
├── ai-dlc-sb-code-review\       SKILL.md
├── ai-dlc-sb-code-revise\       SKILL.md
├── ai-dlc-sb-springboot-guide\  SKILL.md
├── ai-dlc-sb-anyframe-guide\    SKILL.md
├── ai-dlc-sb-mybatis-guide\     SKILL.md
├── ai-dlc-sb-dbio-guide\        SKILL.md
├── ai-dlc-sb-security-guide\    SKILL.md
├── ai-dlc-sb-db-guide\          SKILL.md
└── ai-dlc-sb-liquibase-guide\   SKILL.md
```

**총 파일: SKILL.md 23개 + template.md 9개 = 32개**

---

## 스킬별 핵심 설계

| # | 스킬명 | 트리거 (대표) | 유형 | 산출물 |
|:---:|:---|:---|:---:|:---|
| 1 | project-setup | "pom.xml 만들어줘" | create | pom.xml, application-*.yml |
| 2 | anyframe-setup | "Anyframe 설정해줘" | create | AnyframeConfig.java, 설정 일체 |
| 3 | migration-plan | "DB 마이그레이션 계획" | create | DB마이그레이션계획_*.md |
| 4 | sql-plan | "SQL 생성 계획" | create | SQL생성계획_*.md |
| 5 | sql-gen | "DDL 만들어줘" | create | DDL/DML .sql 파일 |
| 6 | migration-exec | "changeset 만들어줘" | create | changelog.xml / V*.sql |
| 7 | layer-impl | "전체 코드 생성해줘" | create(orch) | 레이어 실행 안내 |
| 8 | vo-gen | "VO 코드 생성" | create | *VO.java, *ReqVO.java |
| 9 | mapper-gen | "MyBatis Mapper 만들어줘" | create | *Mapper.java + XML |
| 10 | service-gen | "Service 코드 생성" | create | *Service.java + Impl |
| 11 | controller-gen | "Controller 코드 생성" | create | *Controller.java |
| 12 | unit-test-gen | "JUnit 테스트 만들어줘" | create | *Test.java |
| 13 | unit-test-validate | "테스트 코드 검토" | validate | 테스트코드_검증_*.md |
| 14 | unit-test-revise | "테스트 코드 수정" | revise | 수정된 *Test.java |
| 15 | code-review | "코드 검토해줘" | validate | 코드품질검토_*.md |
| 16 | code-revise | "리뷰 결과 반영" | revise | 수정된 소스코드 |
| 17 | springboot-guide | "Spring Boot 가이드" | guide | 대화창 출력 |
| 18 | anyframe-guide | "Anyframe 사용법" | guide | 대화창 출력 |
| 19 | mybatis-guide | "MyBatis 쿼리 작성법" | guide | 대화창 출력 |
| 20 | dbio-guide | "DBIO 사용법" | guide | 대화창 출력 |
| 21 | security-guide | "SQL Injection 방지" | guide | 대화창 출력 |
| 22 | db-guide | "DB 개발 가이드" | guide | 대화창 출력 |
| 23 | liquibase-guide | "Liquibase 가이드" | guide | 대화창 출력 |

---

## 검증 방법

| 트리거 문장 | 기대 스킬 |
|:---|:---|
| "Spring Boot 프로젝트 설정해줘" | `ai-dlc-sb-project-setup` |
| "Anyframe 프로젝트 생성해줘" | `ai-dlc-sb-anyframe-setup` |
| "DB 마이그레이션 전략 세워줘" | `ai-dlc-sb-migration-plan` |
| "DDL SQL 만들어줘" | `ai-dlc-sb-sql-gen` |
| "VO 코드 생성해줘" | `ai-dlc-sb-vo-gen` |
| "MyBatis Mapper 만들어줘" | `ai-dlc-sb-mapper-gen` |
| "Service 코드 만들어줘" | `ai-dlc-sb-service-gen` |
| "Controller 코드 생성해줘" | `ai-dlc-sb-controller-gen` |
| "JUnit 테스트 만들어줘" | `ai-dlc-sb-unit-test-gen` |
| "생성된 코드 리뷰해줘" | `ai-dlc-sb-code-review` |
| "MyBatis 쿼리 어떻게 써?" | `ai-dlc-sb-mybatis-guide` |
| "SQL Injection 방지 방법은?" | `ai-dlc-sb-security-guide` |
| "Liquibase changeset 작성법은?" | `ai-dlc-sb-liquibase-guide` |

---

## 비범위

- 프론트엔드 코드 생성 (Vue/React 등)
- CI/CD 파이프라인 설정
- 컨테이너(Docker/Kubernetes) 설정
- 성능 테스트 코드 생성
- 실제 빌드·배포 실행
- 통합 테스트 코드 생성 (별도 PLAN으로 관리)
