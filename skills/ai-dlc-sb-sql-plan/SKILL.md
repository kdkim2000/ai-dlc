---
name: ai-dlc-sb-sql-plan
description: AI-DLC 개발단계(Spring Boot) 스킬. ANSI 기반 DDL/DML SQL 생성 계획을 수립한다. "SQL 생성 계획", "ANSI SQL 계획", "DDL 설계", "SQL 파일 구조 계획", "쿼리 계획 수립", "DDL 파일 계획", "SQL 작성 계획" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC ANSI SQL 생성 계획

데이터설계서를 기반으로 **DDL/DML SQL 파일 생성 계획서**를 작성한다. 테이블 생성 순서, 파일 분류, 명명 규칙을 확정하여 `ai-dlc-sb-sql-gen` 스킬 실행의 기초를 제공한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "SQL 생성 계획", "ANSI SQL 계획", "DDL 설계"
- "SQL 파일 구조 계획", "쿼리 계획 수립", "DDL 파일 계획", "SQL 작성 계획"

## 입력

- **필수**: 데이터설계서 (`ai-dlc-data-design` 산출물) 또는 테이블 목록
- **선택**: DB 마이그레이션 계획 (`ai-dlc-sb-migration-plan` 산출물), DB 종류

## 분석 절차

1. **테이블 목록 파악**: 데이터설계서에서 테이블명·관계 추출
2. **FK 의존성 분석**: 부모-자식 관계 그래프 → 위상 정렬로 생성 순서 결정
3. **SQL 파일 분류 계획**:
   - DDL: CREATE TABLE, 인덱스, 제약 조건
   - DML: 기준 데이터(공통 코드, 초기 마스터 데이터)
   - REF: 테스트용 샘플 데이터 (dev 환경 전용)
4. **SQL 파일 명명 규칙 확정**:
   - DDL 파일: `{NNN}_{테이블명}.sql` (생성 순서 번호 포함)
   - 통합 파일: `schema_all.sql` (전체 DDL 통합본)
5. **DB 방언 처리 계획**: ANSI SQL 기본, DB별 타입 매핑 목록 정리
6. **계획서 문서 생성**

## DB 타입 매핑 기준 (ANSI → DB 방언)

| ANSI 타입 | MySQL | PostgreSQL | Oracle |
|:---|:---|:---|:---|
| INTEGER | INT | INTEGER | NUMBER(10) |
| BIGINT | BIGINT | BIGINT | NUMBER(19) |
| VARCHAR(N) | VARCHAR(N) | VARCHAR(N) | VARCHAR2(N) |
| TEXT | LONGTEXT | TEXT | CLOB |
| DATETIME | DATETIME | TIMESTAMP | DATE |
| BOOLEAN | TINYINT(1) | BOOLEAN | NUMBER(1) |

## 산출물

파일명: `SQL생성계획_{사업명}_{YYYYMMDD}.md` (`${CLAUDE_SKILL_DIR}/template.md` 사용)
