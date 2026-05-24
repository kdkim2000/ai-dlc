---
name: ai-dlc-sb-anyframe-setup
description: AI-DLC 개발단계(Spring Boot) 스킬. Anyframe 프레임워크를 Spring Boot 프로젝트에 설정한다. "Anyframe 프로젝트 생성", "Anyframe 설정", "애니프레임 설정해줘", "Anyframe 초기화", "Anyframe 연동", "애니프레임 적용" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Anyframe 프레임워크 프로젝트 설정

Spring Boot 프로젝트에 **Anyframe 공통 프레임워크**를 적용하기 위한 의존성·설정 파일·공통 컴포넌트를 생성한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "Anyframe 프로젝트 생성", "Anyframe 설정", "애니프레임 설정해줘", "Anyframe 초기화"
- "Anyframe 연동", "애니프레임 적용"
- Spring Boot 프로젝트에 Anyframe을 추가하라는 요청

## 입력

- **필수**: Spring Boot 프로젝트 구조 (`ai-dlc-sb-project-setup` 산출물 또는 기존 프로젝트)
- **선택**: Anyframe 버전, 사용할 Anyframe 컴포넌트 목록

## 분석 절차

1. **프로젝트 구조 파악**: 기존 pom.xml/build.gradle 확인
2. **Anyframe 의존성 추가**:
   - anyframe-core 의존성 추가
   - 필요 컴포넌트별 의존성 추가 (anyframe-query-plugin, anyframe-pagination-plugin 등)
3. **Anyframe 설정 파일 생성**:
   - applicationContext.xml 또는 Java Config 클래스
   - 공통 예외 처리 설정
   - 트랜잭션 매니저 설정
4. **공통 컴포넌트 설정**:
   - 공통 응답 포맷 클래스
   - 공통 예외 처리 클래스 (@ControllerAdvice)
   - 페이지네이션 설정
5. **연동 검증 체크리스트** 출력

## 설계 원칙

- Anyframe 설정은 Spring Boot 자동 설정과 충돌하지 않도록 주의
- XML 설정과 Java Config 중 프로젝트 일관성에 맞는 방식 선택
- 미확인 항목: `# TODO: Anyframe 버전 확인 필요` 주석 필수

## 산출물

- `pom.xml` 또는 `build.gradle` 수정 (Anyframe 의존성 추가)
- `src/main/java/.../config/AnyframeConfig.java` 또는 `applicationContext-anyframe.xml`
- `src/main/java/.../exception/GlobalExceptionHandler.java`
- `src/main/java/.../vo/ApiResponse.java` (공통 응답 포맷)
- Anyframe 설정 검증 체크리스트 (`설정검증_체크리스트_{YYYYMMDD}.md`)
