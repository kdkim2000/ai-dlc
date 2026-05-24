---
name: ai-dlc-data-design
description: AI-DLC 설계단계 스킬. 유즈케이스·요구사항 기반으로 논리/물리 ERD와 테이블 정의서를 생성한다. "데이터 설계서 만들어줘", "DB 설계해줘", "ERD 설계", "테이블 설계해줘", "논리 ERD 만들어줘", "물리 ERD 생성", "DDL 초안 만들어줘", "데이터 모델 설계", "엔터티 설계" 같은 표현이 나오면 반드시 이 스킬을 사용하라. 소스코드가 아닌 요구사항·UC 기반 신규 설계일 때 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC 데이터 설계서 생성

유즈케이스·요구사항을 기반으로 **논리 ERD + 물리 ERD + 테이블 정의서 + DDL 초안**을 생성한다.

> `ai-dlc-data-model-analysis`(소스코드/DDL 역공학)와 구분: 이 스킬은 **요구사항/UC 기반 신규 설계** 전용이다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "데이터 설계서 만들어줘", "DB 설계해줘", "ERD 설계", "테이블 설계해줘"
- "논리 ERD 만들어줘", "물리 ERD 생성", "DDL 초안 만들어줘"
- "데이터 모델 설계", "엔터티 설계"
- UC 문서·요구사항을 주며 "DB로 변환해줘"라고 할 때

## 입력

- **필수**: 아래 중 1개 이상
  - 유즈케이스 문서 (`ai-dlc-usecase-create` 산출물)
  - 요구사항 정의서 (FR 목록)
- **선택**: 도메인 용어사전, 데이터 모델 분석서 (레거시 참조), DB 종류 지정 (MySQL/PostgreSQL/Oracle 등)

## 분석 절차

1. **엔터티 후보 도출**: UC/FR에서 명사형 도메인 객체 추출
   - UC 액터, 기본 흐름의 데이터 객체, FR의 관리 대상
   - 도메인 용어사전 참조로 표준 엔터티명 적용
2. **관계 정의**: 1:1, 1:N, M:N 관계 결정
   - M:N → 연관 엔터티(관계 테이블)로 분해
3. **논리 ERD 설계**: Mermaid `erDiagram`, 3NF 준수
   - 식별자(PK) 결정, 속성 정의, 외래키(FK) 관계
4. **물리 ERD 설계**: DB 타입 매핑, 인덱스 결정
   - 논리 타입 → DB 타입 (VARCHAR, INT, DATETIME 등)
   - 인덱스 후보: 자주 조회하는 FK 컬럼, 검색 조건 컬럼
5. **테이블 정의서 작성**: 테이블별 컬럼 상세
6. **DDL 초안 생성**: CREATE TABLE 구문 (DB 종류 미지정 시 표준 SQL)
7. **데이터 표준 정리**: 공통 코드, 날짜 포맷, 명명 규칙

## 설계 원칙

- **3NF 준수**: 반복 그룹 제거(1NF) → 부분 종속 제거(2NF) → 이행 종속 제거(3NF)
- **PK 전략**: 대리키(surrogate key, AUTO_INCREMENT/SEQUENCE) 우선
- **명명 규칙**: 테이블명 `TB_대문자_스네이크케이스`, 컬럼명 `소문자_스네이크케이스`
- **공통 컬럼**: `created_at`, `updated_at`, `created_by` 모든 테이블 기본 포함
- **미확인 항목**: `-- TODO: 확인 필요` 주석 필수

## 산출물 포맷

파일명: `데이터설계서_{사업명}_{YYYYMMDD}.md` (`${CLAUDE_SKILL_DIR}/template.md` 사용)
