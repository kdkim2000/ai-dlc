---
name: ai-dlc-sequence-design
description: AI-DLC 설계단계 스킬. 유즈케이스·API 설계서 기반으로 Mermaid 시퀀스 다이어그램을 생성한다. "시퀀스 다이어그램 만들어줘", "흐름 다이어그램 만들어줘", "객체 상호작용 설계", "sequence diagram 작성", "호출 순서 다이어그램", "UC 시나리오를 시퀀스로 변환" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC 시퀀스 다이어그램 생성

유즈케이스·API 설계서를 기반으로 **Mermaid sequenceDiagram + 메시지 정의 표 + UC-시퀀스 연계 매핑**을 생성한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "시퀀스 다이어그램 만들어줘", "흐름 다이어그램 만들어줘", "객체 상호작용 설계"
- "sequence diagram 작성", "호출 순서 다이어그램"
- "UC 시나리오를 시퀀스로 변환"
- UC 문서·API 설계서를 주며 "시퀀스로 그려줘"라고 할 때

## 입력

- **필수**: 아래 중 1개 이상
  - 유즈케이스 문서 (`ai-dlc-usecase-create` 산출물)
  - API 설계서 (`ai-dlc-api-design` 산출물)
- **선택**: 클래스 설계서 (참여자 결정용), 특정 시나리오 지정

## 분석 절차

1. **시퀀스 대상 UC 선정**: 
   - UC 문서 전체 시: 복잡도 높은 UC(대안/예외 흐름 多) 우선
   - 특정 UC 지정 시: 해당 UC만 처리
2. **참여자 결정**:
   - Actor: UC의 주 액터
   - Controller: API operationId 기반 또는 기본값
   - Service: 비즈니스 로직 처리 레이어
   - DB/Repository: 데이터 접근 레이어
   - External: 외부 시스템/API 연동
3. **SEQ-NNN 채번**: 001부터 순차 부여
4. **기본 흐름 → 시퀀스 변환**: UC 기본 흐름 단계를 메시지 교환으로 변환
5. **대안/예외 흐름**: `alt`, `opt`, `loop` 블록으로 분기 표현
6. **Mermaid sequenceDiagram 생성**: 주석으로 흐름 단계 번호 포함
7. **메시지 정의 표 작성**: 각 메시지의 의미·파라미터·응답 정의
8. **UC-시퀀스 연계 매핑 표 작성**

## 다이어그램 작성 원칙

- 참여자는 역할 명칭 사용: `Actor`, `Controller`, `Service`, `Repository`, `ExternalAPI`
- 동기 메시지: `->>` (응답 필요)
- 응답 메시지: `-->>` (점선 반환)
- 비동기: `--)` (응답 불필요)
- `activate/deactivate`로 처리 구간 명시
- `Note over`로 중요 처리 내용 주석

## 산출물 포맷

파일명: `시퀀스다이어그램_{사업명}_{YYYYMMDD}.md` (`${CLAUDE_SKILL_DIR}/template.md` 사용)

## 엣지 케이스

- **단일 UC 지정**: 해당 UC의 기본+대안+예외 흐름 모두 하나의 다이어그램에 표현
- **외부 시스템 연동**: External 참여자 별도 박스, API 응답 시간 `Note` 주석
- **비동기 메시지 큐**: 이벤트 퍼블리시 → 컨슈머 구조 표현
- **페이지네이션/목록 조회**: `loop` 블록으로 반복 처리 표현
