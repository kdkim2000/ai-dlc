---
name: ai-dlc-sb-mapper-gen
description: AI-DLC 개발단계(Spring Boot) 스킬. Mapper 레이어 코드를 생성한다(MyBatis/DBIO). "Mapper 코드 생성", "MyBatis Mapper 만들어줘", "DBIO 생성", "매퍼 코드 생성", "MyBatis XML 만들어줘", "매퍼 인터페이스 생성", "쿼리 매퍼 만들어줘" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Mapper 레이어 코드 생성 (MyBatis/DBIO)

VO 클래스와 데이터설계서를 기반으로 **Java Mapper 인터페이스 + MyBatis XML 매핑 파일**을 실제 생성한다. BXCM DBIO 방식 선택 시 DBIO 규칙에 따라 생성한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "Mapper 코드 생성", "MyBatis Mapper 만들어줘", "DBIO 생성"
- "매퍼 코드 생성", "MyBatis XML 만들어줘", "매퍼 인터페이스 생성"

## 입력

- **필수**: VO 클래스 (`ai-dlc-sb-vo-gen` 산출물) 또는 데이터설계서
- **선택**: DBIO 방식 지정, 커스텀 쿼리 요구사항

## 분석 절차

1. **Mapper 방식 결정**: MyBatis (기본) / BXCM DBIO (지정 시)
2. **인터페이스 메서드 목록 결정** (CRUD 기본):
   - `selectList({도메인명}SearchVO)` → 목록 조회
   - `selectOne(Long id)` → 단건 조회
   - `insert({도메인명}VO)` → 등록
   - `update({도메인명}VO)` → 수정
   - `delete(Long id)` → 삭제
   - `count({도메인명}SearchVO)` → 전체 건수
3. **XML 매핑 파일 생성**:
   - namespace: Mapper 인터페이스 FQCN
   - resultMap: 테이블 컬럼 → VO 필드 매핑
   - 동적 쿼리: `<where>`, `<if>`, `<foreach>` 활용
   - 공통 컬럼 자동 처리: `created_at = NOW()`, `created_by = #{createdBy}`
4. **N+1 방지**: 복합 조회는 JOIN 쿼리로 설계
5. **파일 저장**: 실제 경로에 파일 생성

## MyBatis 쿼리 원칙

- **파라미터 바인딩**: `#{}` 사용 (PreparedStatement), `${}` 금지 (SQL Injection 위험)
- **동적 정렬**: `ORDER BY` 절에 `${sortBy}` 사용 시 화이트리스트 검증 필수 주석
- **resultMap 우선**: `resultType` 대신 `resultMap` 사용으로 타입 안전성 확보
- **페이징**: LIMIT/OFFSET 기반 (`#{offset}`, `#{size}`)

## BXCM DBIO 방식 (지정 시)

- 쿼리 ID 규칙: `{namespace}.{동작}_{설명}` (예: `userMapper.selectUserList`)
- 입출력 VO: 단일 VO 클래스로 입출력 통일
- XML 파일 위치: `src/main/resources/sql/{도메인명}Mapper.xml`

## 산출물

- `src/main/java/{basePackage}/mapper/{도메인명}Mapper.java`
- `src/main/resources/mapper/{도메인명}Mapper.xml`
