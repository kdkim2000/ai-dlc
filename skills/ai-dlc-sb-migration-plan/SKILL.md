---
name: ai-dlc-sb-migration-plan
description: AI-DLC 개발단계(Spring Boot) 스킬. Liquibase/Flyway DB 마이그레이션 전략을 계획한다. "DB 마이그레이션 계획", "Liquibase 설정 계획", "Flyway 설정 계획", "DB 버전 관리 전략", "마이그레이션 전략 수립", "DB 변경 관리 계획" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC DB 마이그레이션 전략 계획

데이터설계서를 기반으로 **Liquibase/Flyway DB 마이그레이션 전략 계획서**를 생성한다. 도구 선정, 버전 체계, 롤백 전략, 환경별 적용 계획을 포함한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "DB 마이그레이션 계획", "Liquibase 설정 계획", "Flyway 설정 계획"
- "DB 버전 관리 전략", "마이그레이션 전략 수립", "DB 변경 관리 계획"

## 입력

- **필수**: 데이터설계서 (`ai-dlc-data-design` 산출물) 또는 테이블 목록
- **선택**: 마이그레이션 도구 선택 (Liquibase 기본 / Flyway 선택), DB 종류, 환경 수

## 분석 절차

1. **도구 선택 결정**: 미지정 시 **Liquibase 기본** 적용
   - Liquibase: XML changeset 방식, 다중 DB 지원, 롤백 명시적 지원
   - Flyway: SQL 파일 기반, 단순하고 빠름, Spring Boot 자동 설정 용이
2. **버전 번호 체계 결정**:
   - Liquibase: `V{YYYYMMDD}_{NNN}_{설명}` 형태 changeSet id
   - Flyway: `V{N}__{설명}.sql` (예: `V1__create_user_table.sql`)
3. **변경 관리 프로세스**: 브랜치별 마이그레이션 파일 관리, 충돌 방지 규칙
4. **롤백 전략**: Liquibase rollback 태그 / Flyway Undo 마이그레이션
5. **환경별 적용 계획**: dev → stg → prod 순서, 데이터 초기화 전략
6. **테이블 생성 순서**: FK 의존성 위상 정렬 (부모 테이블 먼저)
7. **계획서 문서 생성**

## 산출물

파일명: `DB마이그레이션계획_{사업명}_{YYYYMMDD}.md` (`${CLAUDE_SKILL_DIR}/template.md` 사용)
