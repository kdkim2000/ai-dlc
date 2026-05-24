---
name: ai-dlc-sb-project-setup
description: AI-DLC 개발단계(Spring Boot) 스킬. Spring Boot 프로젝트 초기 설정 파일을 생성한다. "Spring Boot 프로젝트 설정", "스프링부트 프로젝트 생성", "pom.xml 만들어줘", "application.yml 설정", "프로젝트 초기 설정", "스프링부트 초기화", "Maven 설정 만들어줘", "Gradle 설정 만들어줘" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Spring Boot 프로젝트 초기 설정

설계단계 산출물 기반으로 Spring Boot 프로젝트의 **빌드 설정(pom.xml/build.gradle) + 애플리케이션 설정(application.yml) + 기본 패키지 구조**를 생성한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "Spring Boot 프로젝트 설정", "스프링부트 프로젝트 생성", "프로젝트 초기 설정"
- "pom.xml 만들어줘", "application.yml 설정", "스프링부트 초기화"
- "Maven 설정 만들어줘", "Gradle 설정 만들어줘"

## 입력

- **필수**: 사업명, Java 버전, Spring Boot 버전
- **선택**: DB 종류(MySQL/PostgreSQL/Oracle), 빌드 도구(Maven 기본/Gradle), API 설계서, 클래스설계서

## 분석 절차

1. **기술 스택 확인**: Java 버전·Spring Boot 버전·DB 종류·빌드 도구 결정
   - 미지정 시: Java 17, Spring Boot 3.x, Maven, MySQL 기본 적용
2. **빌드 설정 파일 생성**:
   - Maven: `pom.xml` (parent spring-boot-starter-parent, 핵심 의존성)
   - Gradle: `build.gradle` (plugins, dependencies)
   - 공통 의존성: spring-boot-starter-web, spring-boot-starter-data-jpa 또는 MyBatis, Lombok, spring-boot-starter-validation, spring-boot-starter-test
3. **application.yml 생성**: 기본 설정 + 프로파일별 분리
   - `application.yml` — 공통 설정 (server.port, 로깅 레벨)
   - `application-dev.yml` — 개발 환경 (로컬 DB)
   - `application-stg.yml` — 스테이징 환경
   - `application-prod.yml` — 운영 환경 (민감 정보는 `${ENV_VAR}` 환경 변수 참조)
4. **패키지 구조 생성**: 표준 레이어드 디렉터리 생성
   ```
   src/main/java/{basePackage}/
   ├── controller/
   ├── service/
   ├── mapper/
   ├── vo/
   └── config/
   src/main/resources/
   ├── mapper/      (MyBatis XML)
   └── db/
   src/test/java/{basePackage}/
   ```
5. **보안 설정**: 민감 정보(DB 패스워드, API 키) → 환경 변수 참조 패턴 적용
6. **README.md 초안**: 프로젝트 설명·실행 방법·환경 설정 가이드

## 설계 원칙

- **민감 정보 금지**: `application-*.yml`에 패스워드/시크릿 값 하드코딩 금지 → `${DB_PASSWORD}` 형태
- **프로파일 분리**: dev/stg/prod 환경별 설정 파일 분리 필수
- **미확인 항목**: `# TODO: 확인 필요` 주석 필수
- **버전 고정**: 의존성 버전은 parent BOM 또는 명시적 버전으로 고정

## 산출물

파일명: 프로젝트 루트에 직접 저장 (`${CLAUDE_SKILL_DIR}/template.md` 구조 참조)
- `pom.xml` 또는 `build.gradle`
- `src/main/resources/application.yml` + `application-{profile}.yml`
- 기본 패키지 디렉터리 구조
- `README.md` 초안
