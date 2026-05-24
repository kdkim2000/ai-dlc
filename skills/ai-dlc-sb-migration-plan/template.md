# DB 마이그레이션 계획서

| 항목 | 내용 |
|:---|:---|
| 사업명 | {{사업명}} |
| 마이그레이션 도구 | {{도구명}} (Liquibase / Flyway) |
| DB 종류 | {{DB종류}} |
| 테이블 수 | {{테이블수}}개 |
| 작성 일시 | {{작성일시}} |

---

## 1. 도구 선택 및 근거

| 항목 | Liquibase | Flyway |
|:---|:---|:---|
| 파일 형식 | XML/YAML/JSON/SQL changeset | SQL 파일 (V{N}__*.sql) |
| 롤백 | 명시적 rollback 태그 지원 | Flyway Teams 이상 (유료) |
| 다중 DB | 광범위 지원 | 주요 DB 지원 |
| Spring Boot 통합 | 자동 설정 지원 | 자동 설정 지원 |

**선택: {{도구명}}** — {{선택 근거}}

---

## 2. 버전 번호 체계

{{#Liquibase}}
| 구성 요소 | 규칙 | 예시 |
|:---|:---|:---|
| changeSet id | `{YYYYMMDD}-{NNN}` | `20250523-001` |
| author | 작성자 ID | `dev_team` |
| 파일 구분 | DDL / DML / REF (기준 데이터) | — |
{{/Liquibase}}

{{#Flyway}}
| 구성 요소 | 규칙 | 예시 |
|:---|:---|:---|
| 버전 접두사 | `V{N}__` | `V1__` |
| 설명 | 언더스코어 구분, 영문 소문자 | `create_user_table` |
| 전체 파일명 | `V{N}__{설명}.sql` | `V1__create_user_table.sql` |
{{/Flyway}}

---

## 3. 테이블 생성 순서 (FK 의존성 위상 정렬)

| 순서 | 테이블명 | 의존 테이블 | 비고 |
|:---:|:---|:---|:---|
{{테이블순서_ROWS}}

---

## 4. 마이그레이션 파일 목록

| 순서 | 파일명 | 내용 | 유형 |
|:---:|:---|:---|:---:|
{{마이그레이션파일_ROWS}}

---

## 5. 환경별 적용 계획

| 환경 | 적용 대상 | 적용 방식 | 주의사항 |
|:---|:---|:---|:---|
| dev | 전체 DDL + DML | 자동 (Spring Boot 기동 시) | 스키마 재생성 허용 |
| stg | 증분 changeset | 수동 검토 후 자동 | 운영 데이터 유사 환경 |
| prod | 증분 changeset | 배포 전 DBA 검토 필수 | 롤백 계획 사전 수립 |

---

## 6. 롤백 전략

{{#Liquibase}}
```xml
<!-- rollback 예시 -->
<changeSet id="20250523-001" author="dev_team">
    <createTable tableName="TB_USER">...</createTable>
    <rollback>
        <dropTable tableName="TB_USER"/>
    </rollback>
</changeSet>
```
{{/Liquibase}}

{{#Flyway}}
- Flyway Community: SQL 롤백 불가 → 새 마이그레이션 파일로 변경 취소
- `V{N+1}__rollback_xxx.sql` 방식으로 역변환 SQL 수동 작성
{{/Flyway}}

---

## 문서 버전 이력

| 버전 | 일자 | 작성자 | 변경 내용 |
|:---|:---|:---|:---|
| v0.1 | {{작성일시}} | 초안 작성 | 최초 생성 |
