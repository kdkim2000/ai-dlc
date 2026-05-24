---
name: ai-dlc-sb-sql-gen
description: AI-DLC 개발단계(Spring Boot) 스킬. 데이터설계서 기반으로 ANSI SQL DDL/DML 파일을 생성한다. "SQL 파일 생성", "DDL 만들어줘", "DML 생성", "테이블 생성 SQL", "SQL 스크립트 만들어줘", "CREATE TABLE 만들어줘", "인덱스 SQL 생성" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC SQL DDL/DML 파일 생성

데이터설계서를 기반으로 **ANSI SQL DDL(테이블 생성·인덱스·제약) + DML(기준 데이터) 파일**을 실제로 생성한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "SQL 파일 생성", "DDL 만들어줘", "DML 생성", "테이블 생성 SQL"
- "SQL 스크립트 만들어줘", "CREATE TABLE 만들어줘", "인덱스 SQL 생성"

## 입력

- **필수**: 데이터설계서 (`ai-dlc-data-design` 산출물)
- **선택**: SQL 생성 계획 (`ai-dlc-sb-sql-plan` 산출물), DB 종류

## 분석 절차

1. **테이블 생성 순서 결정**: FK 의존성 위상 정렬 (부모 테이블 우선)
2. **DDL 생성**:
   - `CREATE TABLE`: 컬럼명·타입·NOT NULL·DEFAULT·PK 제약 포함
   - 공통 컬럼 자동 추가: `created_at DATETIME DEFAULT CURRENT_TIMESTAMP`, `updated_at DATETIME ON UPDATE CURRENT_TIMESTAMP`, `created_by VARCHAR(50)`
   - 외래키 제약: `ALTER TABLE ... ADD CONSTRAINT FK_...`
   - 인덱스: FK 컬럼 + 자주 조회되는 컬럼
3. **DML 생성**: 기준 데이터(공통 코드, 초기 마스터) INSERT 문
4. **DB 방언 처리**: DB 종류 미지정 시 ANSI SQL 기본, DB별 분기
5. **파일 저장**: SQL 생성 계획의 파일 구조에 따라 저장

## 생성 원칙

- **ANSI SQL 우선**: 특정 DB 구문은 주석으로 DB 종류 명시
- **DROP 구문 금지**: 운영 환경 실수 방지 (dev 전용 파일에만 허용, 명시 주석 필수)
- **주석 필수**: 각 테이블 생성 구문 위에 테이블 한글명·설명 주석
- **문자셋**: `CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci` (MySQL 기준)

## 산출물

- `src/main/resources/db/sql/{NNN}_{테이블명}.sql` — 테이블별 DDL
- `src/main/resources/db/sql/schema_all.sql` — 전체 통합 DDL
- `src/main/resources/db/sql/dml_*.sql` — 기준 데이터 DML
- `src/main/resources/db/sql/ref_*.sql` — dev용 샘플 데이터 (선택)

## 엣지 케이스

- **M:N 관계 테이블**: 연관 엔터티 테이블 자동 생성 (부모 테이블 다음 순서)
- **공통 코드 테이블**: 코드 그룹·코드값·코드명 구조 표준 DDL
- **시퀀스/자동증가**: MySQL AUTO_INCREMENT / PostgreSQL SERIAL / Oracle SEQUENCE
