---
name: ai-dlc-sb-layer-impl
description: AI-DLC 개발단계(Spring Boot) 스킬. 전체 레이어 코드 생성을 오케스트레이션한다. "레이어 코드 구현", "전체 코드 생성해줘", "레이어별 코드 만들어줘", "CRUD 코드 생성", "전체 구현 실행", "전 레이어 구현해줘", "VO부터 Controller까지 만들어줘" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC 전체 레이어 코드 생성 오케스트레이터

클래스설계서·데이터설계서·API설계서를 기반으로 **VO → Mapper → Service → Controller** 순서로 각 레이어 스킬을 안내하고, 생성 완료 체크리스트를 출력한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "레이어 코드 구현", "전체 코드 생성해줘", "레이어별 코드 만들어줘"
- "CRUD 코드 생성", "전체 구현 실행", "VO부터 Controller까지 만들어줘"

## 입력

- **필수**: 클래스설계서 (CLS-NNN), 데이터설계서 (테이블 정의), API설계서 (operationId)
- **선택**: 기존 프로젝트 구조 (패키지명, 기술스택)

## 분석 절차

1. **입력 설계서 파악**: 클래스설계서에서 CLS 목록 추출, 각 클래스의 레이어(Controller/Service/Mapper/VO) 확인
2. **생성 대상 확인**: 어떤 도메인(테이블/엔터티) 의 코드를 생성할지 결정
3. **생성 순서 안내**: 레이어 의존성 기반 순서 안내
4. **순차 실행 가이드**: 각 스킬 호출 시 필요한 입력 설명
5. **완료 체크리스트**: 생성 완료 항목 확인

## 레이어 생성 순서

```
1단계: ai-dlc-sb-vo-gen
  입력: 데이터설계서 (테이블 컬럼) + 클래스설계서 (CLS-NNN Domain/VO)
  출력: src/main/java/.../vo/*.java

2단계: ai-dlc-sb-mapper-gen
  입력: VO 클래스 + 데이터설계서
  출력: src/main/java/.../mapper/*.java
       src/main/resources/mapper/*.xml

3단계: ai-dlc-sb-service-gen
  입력: 클래스설계서 (CLS-NNN Service) + API설계서 + Mapper 인터페이스
  출력: src/main/java/.../service/*.java
       src/main/java/.../service/impl/*Impl.java

4단계: ai-dlc-sb-controller-gen
  입력: API설계서 (operationId, 경로) + Service 인터페이스
  출력: src/main/java/.../controller/*.java
```

## 출력 형식

오케스트레이터 스킬은 실제 코드를 직접 생성하지 않고, 아래 형식으로 안내문을 대화창에 출력한다.

```
## 레이어 구현 실행 계획

대상 도메인: {도메인명}
패키지 기준: {basePackage}

### 실행 순서

| 단계 | 스킬 | 입력 | 예상 산출물 |
|:---:|:---|:---|:---|
| 1 | ai-dlc-sb-vo-gen | 데이터설계서 + CLS-{NNN} | {도메인명}VO.java |
| 2 | ai-dlc-sb-mapper-gen | VO 코드 + 데이터설계서 | {도메인명}Mapper.java + XML |
| 3 | ai-dlc-sb-service-gen | CLS-{NNN} + API설계서 | {도메인명}Service.java + Impl |
| 4 | ai-dlc-sb-controller-gen | API설계서 + Service | {도메인명}Controller.java |

### 생성 완료 체크리스트

- [ ] VO/DTO 생성 완료
- [ ] Mapper 인터페이스 + XML 생성 완료
- [ ] Service 인터페이스 + Impl 생성 완료
- [ ] Controller 생성 완료
- [ ] 단위 테스트 생성 (ai-dlc-sb-unit-test-gen)
- [ ] 코드 품질 검토 (ai-dlc-sb-code-review)
```
