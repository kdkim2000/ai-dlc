---
name: ai-dlc-sb-liquibase-guide
description: AI-DLC 개발단계(Spring Boot) 스킬. Liquibase/Flyway 마이그레이션 가이드라인을 제공한다. "Liquibase 가이드", "Flyway 가이드", "DB 마이그레이션 방법", "마이그레이션 스크립트 작성법", "changeset 작성법", "Liquibase 어떻게 써", "Flyway 버전 관리" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Liquibase/Flyway 마이그레이션 가이드라인

Liquibase changeset 작성 규칙·Flyway 버전 파일 명명·롤백 작성·환경별 적용·팀 협업 충돌 방지를 대화창에 직접 출력한다. 파일을 저장하지 않는다.

## 트리거

- "Liquibase 가이드", "Flyway 가이드", "DB 마이그레이션 방법"
- "마이그레이션 스크립트 작성법", "changeset 작성법", "Flyway 버전 관리"

---

## 가이드 내용 (인라인 내장)

### 1. Liquibase 기본 규칙

#### 파일 구조
```
src/main/resources/db/changelog/
├── db.changelog-master.xml     ← 마스터 (include 목록만)
├── ddl/
│   ├── 001_create_user.xml
│   └── 002_create_order.xml
└── dml/
    └── 010_insert_common_code.xml
```

#### changeSet id 규칙
```
{YYYYMMDD}-{NNN}   예: 20240523-001
```

- **1 changeSet = 1 목적**: 테이블 생성, 컬럼 추가, 인덱스 생성 각각 별도 changeSet
- **author**: `dev_team` (팀 공통) 또는 담당자 ID
- **id 중복 금지**: 동일 id가 2개 이상이면 Liquibase 오류 발생

#### 안전한 changeSet 패턴
```xml
<changeSet id="20240523-001" author="dev_team">
    <!-- precondition: 이미 존재하면 건너뜀 -->
    <preConditions onFail="MARK_RAN">
        <not><tableExists tableName="TB_USER"/></not>
    </preConditions>
    <createTable tableName="TB_USER">
        <column name="user_id" type="BIGINT" autoIncrement="true">
            <constraints primaryKey="true"/>
        </column>
        <column name="user_nm" type="VARCHAR(100)">
            <constraints nullable="false"/>
        </column>
    </createTable>
    <rollback>
        <dropTable tableName="TB_USER"/>
    </rollback>
</changeSet>
```

### 2. Flyway 기본 규칙

#### 파일명 규칙
```
V{버전}__{설명}.sql
예: V1__create_user_table.sql
    V2__add_email_column.sql
    V10__insert_common_codes.sql
```

- **버전은 연속**해야 함: V1 → V2 → V3
- **버전 중간 삽입 불가**: V1 다음에 V1_5 등 불가
- **설명**: 영문 소문자, 언더스코어 구분
- **수정 금지**: 한 번 배포된 파일은 절대 수정 불가 (checksum 오류)

#### Flyway SQL 파일 예시
```sql
-- V1__create_user_table.sql
CREATE TABLE IF NOT EXISTS TB_USER (
    user_id     BIGINT          NOT NULL AUTO_INCREMENT,
    user_nm     VARCHAR(100)    NOT NULL,
    created_at  DATETIME        NOT NULL DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT PK_TB_USER PRIMARY KEY (user_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

### 3. 환경별 적용 전략

| 환경 | 적용 방식 | 주의사항 |
|:---|:---|:---|
| dev | 자동 (Spring Boot 기동 시) | `drop-first: false` 기본 |
| stg | 수동 검토 후 자동 | 운영 유사 데이터 보호 |
| prod | DBA 검토 후 배포 시 적용 | 롤백 계획 사전 수립 |

```yaml
# application-prod.yml
spring:
  liquibase:
    enabled: true
    contexts: prod
  # flyway:
  #   enabled: true
  #   clean-disabled: true  # prod에서 clean 명령 금지
```

### 4. 롤백 전략

```xml
<!-- Liquibase: rollback 태그로 명시적 역변환 -->
<changeSet id="20240523-002" author="dev_team">
    <addColumn tableName="TB_USER">
        <column name="email" type="VARCHAR(200)"/>
    </addColumn>
    <rollback>
        <dropColumn tableName="TB_USER" columnName="email"/>
    </rollback>
</changeSet>

<!-- 롤백 실행: liquibase rollbackCount 1 -->
```

```sql
-- Flyway: 새 마이그레이션으로 역변환
-- V3__rollback_add_email.sql (Flyway Community는 자동 롤백 불가)
ALTER TABLE TB_USER DROP COLUMN email;
```

### 5. 팀 협업 충돌 방지

- **브랜치별 changeSet id 범위 예약**: feature/A → `20240523-1xx`, feature/B → `20240523-2xx`
- **병합 전 id 충돌 확인**: `grep -r "changeSet id" db/changelog/`
- **절대 수정 금지**: 이미 다른 환경에 배포된 changeSet은 수정 불가 → 새 changeSet 추가
- **테스트**: `liquibase validate` 명령으로 changelog 무결성 사전 검증

### 6. Liquibase vs Flyway 선택 기준

| 항목 | Liquibase | Flyway |
|:---|:---|:---|
| 복잡한 롤백 | 명시적 rollback 태그 지원 | Community: 지원 안함 |
| 파일 형식 | XML/YAML/SQL | SQL만 |
| 러닝커브 | 높음 (XML 구조) | 낮음 (SQL 직접 작성) |
| 다중 DB | 광범위 지원 | 주요 DB 지원 |
| 기본 선택 | ✅ 이 프로젝트 기본 | 단순 프로젝트 선택 |
