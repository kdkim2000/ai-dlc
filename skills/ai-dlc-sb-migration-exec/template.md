# 마이그레이션 실행 파일 템플릿

## Liquibase — db.changelog-master.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.20.xsd">

    <!-- DDL: 테이블 생성 -->
    <include file="classpath:db/changelog/ddl/001_create_common_code.xml"/>
    <include file="classpath:db/changelog/ddl/002_create_{{테이블명}}.xml"/>

    <!-- DML: 기준 데이터 -->
    <include file="classpath:db/changelog/dml/010_insert_common_code.xml"/>

</databaseChangeLog>
```

---

## Liquibase — 테이블 생성 changeset

```xml
<?xml version="1.0" encoding="UTF-8"?>
<databaseChangeLog
    xmlns="http://www.liquibase.org/xml/ns/dbchangelog"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.liquibase.org/xml/ns/dbchangelog
        http://www.liquibase.org/xml/ns/dbchangelog/dbchangelog-4.20.xsd">

    <changeSet id="{{YYYYMMDD}}-001" author="dev_team">
        <comment>{{테이블_한글명}} 테이블 생성</comment>
        <preConditions onFail="MARK_RAN">
            <not><tableExists tableName="{{테이블명}}"/></not>
        </preConditions>
        <createTable tableName="{{테이블명}}" remarks="{{테이블_한글명}}">
            <column name="{{pk컬럼명}}" type="BIGINT" autoIncrement="true" remarks="{{pk설명}}">
                <constraints primaryKey="true" nullable="false"/>
            </column>
            <column name="{{컬럼명}}" type="VARCHAR(100)" remarks="{{설명}}">
                <constraints nullable="false"/>
            </column>
            <column name="{{컬럼명2}}" type="VARCHAR(500)" remarks="{{설명2}}"/>
            <column name="created_at" type="DATETIME" defaultValueComputed="CURRENT_TIMESTAMP" remarks="생성일시">
                <constraints nullable="false"/>
            </column>
            <column name="updated_at" type="DATETIME" remarks="수정일시"/>
            <column name="created_by" type="VARCHAR(50)" remarks="생성자ID"/>
        </createTable>
        <rollback>
            <!-- dev 환경에서만 실행 -->
            <dropTable tableName="{{테이블명}}"/>
        </rollback>
    </changeSet>

    <!-- 인덱스 생성 -->
    <changeSet id="{{YYYYMMDD}}-002" author="dev_team">
        <createIndex tableName="{{테이블명}}" indexName="IX_{{테이블명}}_{{컬럼명}}">
            <column name="{{컬럼명}}"/>
        </createIndex>
        <rollback>
            <dropIndex tableName="{{테이블명}}" indexName="IX_{{테이블명}}_{{컬럼명}}"/>
        </rollback>
    </changeSet>

</databaseChangeLog>
```

---

## Liquibase — DML changeset (기준 데이터)

```xml
<changeSet id="{{YYYYMMDD}}-010" author="dev_team" context="!test">
    <comment>공통코드 기준 데이터 INSERT</comment>
    <insert tableName="TB_COMMON_CODE_GROUP">
        <column name="group_cd" value="{{그룹코드}}"/>
        <column name="group_nm" value="{{그룹명}}"/>
        <column name="use_yn"   value="Y"/>
        <column name="created_by" value="SYSTEM"/>
    </insert>
    <rollback>
        <delete tableName="TB_COMMON_CODE_GROUP">
            <where>group_cd='{{그룹코드}}'</where>
        </delete>
    </rollback>
</changeSet>
```

---

## application.yml — Liquibase 설정

```yaml
spring:
  liquibase:
    enabled: true
    change-log: classpath:db/changelog/db.changelog-master.xml
    contexts: ${SPRING_PROFILES_ACTIVE:dev}  # dev / stg / prod

# application-dev.yml
spring:
  liquibase:
    drop-first: false  # 개발 초기 스키마 재생성 시에만 true 설정
    contexts: dev
```

---

## Flyway — 버전 파일 예시

```sql
-- V1__create_common_code.sql
-- Flyway 마이그레이션: 공통코드 테이블 생성
-- 생성일: {{작성일시}}

CREATE TABLE IF NOT EXISTS TB_COMMON_CODE_GROUP (
    group_cd        VARCHAR(20)     NOT NULL    COMMENT '코드그룹ID',
    group_nm        VARCHAR(100)    NOT NULL    COMMENT '코드그룹명',
    use_yn          CHAR(1)         NOT NULL    DEFAULT 'Y',
    created_at      DATETIME        NOT NULL    DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT PK_TB_COMMON_CODE_GROUP PRIMARY KEY (group_cd)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

## application.yml — Flyway 설정

```yaml
spring:
  flyway:
    enabled: true
    locations: classpath:db/migration
    baseline-on-migrate: true
    baseline-version: 0
    validate-on-migrate: true

# application-dev.yml
spring:
  flyway:
    clean-disabled: false  # dev에서만 flyway:clean 허용
```

---

## pom.xml — 의존성

```xml
<!-- Liquibase -->
<dependency>
    <groupId>org.liquibase</groupId>
    <artifactId>liquibase-core</artifactId>
</dependency>

<!-- Flyway (선택) -->
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-mysql</artifactId>
</dependency>
```
