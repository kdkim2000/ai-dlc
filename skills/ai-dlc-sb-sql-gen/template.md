# SQL DDL/DML 템플릿

## DDL — 테이블 생성 기본 구조

```sql
-- ============================================================
-- 테이블명: {{테이블명}}
-- 설명    : {{테이블_한글명}} — {{테이블_설명}}
-- 작성일  : {{작성일시}}
-- ============================================================
CREATE TABLE {{테이블명}} (
    -- PK
    {{pk컬럼명}}        {{pk타입}}          NOT NULL    COMMENT '{{pk설명}}',

    -- 업무 컬럼
    {{컬럼명}}          {{타입}}            NOT NULL    COMMENT '{{설명}}',
    {{컬럼명2}}         {{타입2}}                       COMMENT '{{설명2}}',

    -- 공통 컬럼 (자동 추가)
    created_at          DATETIME            NOT NULL    DEFAULT CURRENT_TIMESTAMP   COMMENT '생성일시',
    updated_at          DATETIME                        ON UPDATE CURRENT_TIMESTAMP  COMMENT '수정일시',
    created_by          VARCHAR(50)                     COMMENT '생성자ID',

    -- PK 제약
    CONSTRAINT PK_{{테이블명}} PRIMARY KEY ({{pk컬럼명}})
) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  COLLATE=utf8mb4_unicode_ci
  COMMENT='{{테이블_한글명}}';
```

---

## DDL — 인덱스 생성

```sql
-- 단일 컬럼 인덱스
CREATE INDEX IX_{{테이블명}}_{{컬럼명}}
    ON {{테이블명}} ({{컬럼명}});

-- 복합 컬럼 인덱스
CREATE INDEX IX_{{테이블명}}_{{컬럼명1}}_{{컬럼명2}}
    ON {{테이블명}} ({{컬럼명1}}, {{컬럼명2}});

-- 유니크 인덱스
CREATE UNIQUE INDEX UIX_{{테이블명}}_{{컬럼명}}
    ON {{테이블명}} ({{컬럼명}});
```

---

## DDL — 외래키 제약

```sql
-- FK: {{자식테이블}} → {{부모테이블}}
ALTER TABLE {{자식테이블}}
    ADD CONSTRAINT FK_{{자식테이블}}_{{부모테이블}}
    FOREIGN KEY ({{fk컬럼명}})
    REFERENCES {{부모테이블}} ({{pk컬럼명}})
    ON UPDATE RESTRICT
    ON DELETE RESTRICT;
```

---

## DDL — AUTO_INCREMENT PK (MySQL)

```sql
CREATE TABLE {{테이블명}} (
    {{pk컬럼명}}    BIGINT          NOT NULL AUTO_INCREMENT  COMMENT '{{pk설명}}',
    ...
    CONSTRAINT PK_{{테이블명}} PRIMARY KEY ({{pk컬럼명}})
) ENGINE=InnoDB AUTO_INCREMENT=1 ...;
```

---

## DML — 기준 데이터 INSERT

```sql
-- ============================================================
-- 파일명: dml_{{설명}}.sql
-- 내용  : {{기준데이터_설명}}
-- ============================================================

INSERT INTO {{테이블명}} (
    {{컬럼1}}, {{컬럼2}}, {{컬럼3}},
    created_at, created_by
) VALUES
    ('{{값1}}', '{{값2}}', '{{값3}}', NOW(), 'SYSTEM'),
    ('{{값4}}', '{{값5}}', '{{값6}}', NOW(), 'SYSTEM');
```

---

## DML — 공통 코드 테이블 표준 구조

```sql
-- 공통 코드 그룹
CREATE TABLE TB_COMMON_CODE_GROUP (
    group_cd        VARCHAR(20)     NOT NULL    COMMENT '코드그룹ID',
    group_nm        VARCHAR(100)    NOT NULL    COMMENT '코드그룹명',
    group_desc      VARCHAR(500)                COMMENT '코드그룹설명',
    use_yn          CHAR(1)         NOT NULL    DEFAULT 'Y'  COMMENT '사용여부',
    created_at      DATETIME        NOT NULL    DEFAULT CURRENT_TIMESTAMP,
    updated_at      DATETIME                    ON UPDATE CURRENT_TIMESTAMP,
    created_by      VARCHAR(50),
    CONSTRAINT PK_TB_COMMON_CODE_GROUP PRIMARY KEY (group_cd)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='공통코드그룹';

-- 공통 코드
CREATE TABLE TB_COMMON_CODE (
    group_cd        VARCHAR(20)     NOT NULL    COMMENT '코드그룹ID',
    code_cd         VARCHAR(20)     NOT NULL    COMMENT '코드값',
    code_nm         VARCHAR(100)    NOT NULL    COMMENT '코드명',
    code_desc       VARCHAR(500)                COMMENT '코드설명',
    sort_ord        INT             NOT NULL    DEFAULT 0   COMMENT '정렬순서',
    use_yn          CHAR(1)         NOT NULL    DEFAULT 'Y' COMMENT '사용여부',
    created_at      DATETIME        NOT NULL    DEFAULT CURRENT_TIMESTAMP,
    updated_at      DATETIME                    ON UPDATE CURRENT_TIMESTAMP,
    created_by      VARCHAR(50),
    CONSTRAINT PK_TB_COMMON_CODE PRIMARY KEY (group_cd, code_cd),
    CONSTRAINT FK_TB_COMMON_CODE_GROUP
        FOREIGN KEY (group_cd) REFERENCES TB_COMMON_CODE_GROUP(group_cd)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci COMMENT='공통코드';
```

---

## schema_all.sql — 통합 DDL 구조

```sql
-- ============================================================
-- schema_all.sql — 전체 DDL 통합본
-- 프로젝트 : {{사업명}}
-- 생성일시  : {{작성일시}}
-- DB       : {{DB종류}}
-- 주의      : DROP 구문 포함 시 반드시 dev 환경만 실행
-- ============================================================

-- [1] 공통 코드
-- @include: 001_TB_COMMON_CODE_GROUP.sql
-- @include: 002_TB_COMMON_CODE.sql

-- [2] 마스터 테이블
-- @include: 010_TB_{{마스터1}}.sql
-- @include: 011_TB_{{마스터2}}.sql

-- [3] 트랜잭션 테이블
-- @include: 020_TB_{{거래1}}.sql

-- [4] FK 제약 추가
-- @include: 090_FK_CONSTRAINTS.sql

-- [5] 인덱스 생성
-- @include: 091_INDEXES.sql
```

---

## DB 타입 매핑 참조

| ANSI 타입  | MySQL       | PostgreSQL | Oracle       |
|:---------|:------------|:-----------|:-------------|
| INTEGER  | INT         | INTEGER    | NUMBER(10)   |
| BIGINT   | BIGINT      | BIGINT     | NUMBER(19)   |
| VARCHAR(N)| VARCHAR(N) | VARCHAR(N) | VARCHAR2(N)  |
| TEXT     | LONGTEXT    | TEXT       | CLOB         |
| DATETIME | DATETIME    | TIMESTAMP  | DATE         |
| BOOLEAN  | TINYINT(1)  | BOOLEAN    | NUMBER(1)    |
| DECIMAL  | DECIMAL(p,s)| NUMERIC(p,s)| NUMBER(p,s) |
