---
name: ai-dlc-sb-migration-exec
description: AI-DLC 개발단계(Spring Boot) 스킬. Liquibase/Flyway 마이그레이션 설정 파일을 실제 생성한다. "Liquibase 설정 실행", "Flyway 설정해줘", "마이그레이션 실행 파일 만들어줘", "changeset 만들어줘", "마이그레이션 스크립트 생성", "changelog.xml 만들어줘", "Flyway 파일 생성" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC DB 마이그레이션 실행 파일 생성

SQL DDL/DML 파일(ai-dlc-sb-sql-gen 산출물)과 마이그레이션 계획서를 기반으로 **Liquibase changelog + changeset 파일 또는 Flyway 버전 SQL 파일**을 실제 생성하고, `application.yml`에 마이그레이션 설정을 추가한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "Liquibase 설정 실행", "Flyway 설정해줘", "마이그레이션 실행 파일 만들어줘"
- "changeset 만들어줘", "마이그레이션 스크립트 생성", "changelog.xml 만들어줘"

## 입력

- **필수**: SQL 파일 (`ai-dlc-sb-sql-gen` 산출물) 또는 데이터설계서
- **선택**: 마이그레이션 계획서 (`ai-dlc-sb-migration-plan` 산출물), 마이그레이션 도구 지정

## 분석 절차

### Liquibase (기본)

1. **파일 구조 결정**: `src/main/resources/db/changelog/` 하위 계층 구조
2. **master changelog 생성**: `db.changelog-master.xml` (include 목록)
3. **changeset 파일 생성**: 테이블별 또는 버전별 changeset XML
   - changeSet id: `{YYYYMMDD}-{NNN}` 형식
   - author: `dev_team` 기본값
   - rollback 태그: DROP TABLE (dev 환경)
4. **DML changeset 생성**: 기준 데이터 INSERT changeset
5. **application.yml 추가**: Liquibase 설정 블록
6. **pom.xml 확인**: liquibase-core 의존성 추가

### Flyway (선택 지정 시)

1. **파일 구조 결정**: `src/main/resources/db/migration/` 하위
2. **버전 파일 생성**: `V{N}__{설명}.sql` 형식
   - DDL: `V1__create_tables.sql`, `V2__add_indexes.sql`
   - DML: `V10__insert_common_codes.sql`
3. **application.yml 추가**: Flyway 설정 블록
4. **pom.xml 확인**: flyway-core 의존성 추가

## 생성 원칙

- **idempotent 우선**: `IF NOT EXISTS` 사용 또는 precondition 체크
- **순서 보장**: FK 의존성 순서대로 changeset 번호 부여
- **환경 분리**: dev 전용 데이터는 context="dev" 로 분리
- **DROP 금지**: rollback 태그에서만 DROP 사용, precondition 주석 필수

## 산출물 (Liquibase)

```
src/main/resources/db/changelog/
├── db.changelog-master.xml
├── ddl/
│   ├── 001_create_common_code.xml
│   ├── 002_create_{테이블명}.xml
│   └── ...
└── dml/
    ├── 010_insert_common_code.xml
    └── ...
```

## 산출물 (Flyway)

```
src/main/resources/db/migration/
├── V1__create_common_code.sql
├── V2__create_{테이블명}.sql
├── V3__add_indexes.sql
└── V10__insert_common_codes.sql
```
