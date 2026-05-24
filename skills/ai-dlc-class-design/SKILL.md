---
name: ai-dlc-class-design
description: AI-DLC 설계단계 스킬. 유즈케이스·API 설계서 기반으로 UML 클래스 다이어그램과 클래스 설계서를 생성한다. "클래스 설계서 만들어줘", "UML 클래스 다이어그램 작성", "클래스 다이어그램 만들어줘", "도메인 모델 설계", "레이어 구조 설계", "클래스 설계해줘", "패키지 구조 설계" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC 클래스 설계서 생성

유즈케이스·API 설계서를 기반으로 **레이어드 아키텍처 클래스 설계서 + Mermaid classDiagram + 패키지 구조**를 생성한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "클래스 설계서 만들어줘", "UML 클래스 다이어그램 작성", "클래스 다이어그램 만들어줘"
- "도메인 모델 설계", "레이어 구조 설계", "클래스 설계해줘", "패키지 구조 설계"
- UC 문서·API 설계서를 주며 "클래스로 변환해줘"라고 할 때

## 입력

- **필수**: 아래 중 1개 이상
  - 유즈케이스 문서 (`ai-dlc-usecase-create` 산출물)
  - API 설계서 (`ai-dlc-api-design` 산출물)
- **선택**: 데이터 설계서, 기술 스택 정보 (언어/프레임워크)

## 분석 절차

1. **레이어 결정**: 기술 스택 기반 레이어 구성
   - 기본: `Controller → Service → Repository → Domain`
   - 미지정 시 4계층 기본 적용
2. **클래스 후보 도출**:
   - API operationId → Controller 메서드
   - UC 기능 단위 → Service 메서드
   - 엔터티 → Domain 클래스 + Repository 인터페이스
3. **CLS-NNN 채번**: 001부터 순차 부여
4. **속성·메서드·의존관계 정의**:
   - 속성: 접근제어자(+/-/#), 타입, 기본값
   - 메서드: 파라미터, 반환 타입, 접근제어자
   - 관계: 상속(`extends`), 구현(`implements`), 연관(`-->`)
5. **Mermaid classDiagram 생성**: 레이어별 색상 주석 포함
6. **패키지 구조 도식화**: 텍스트 트리 형식

## 설계 원칙

- **레이어 의존 방향**: Controller → Service → Repository → Domain (역방향 금지)
- **SRP 기준**: 단일 클래스 메서드 수 ≤ 10개 (초과 시 분리 검토 주석)
- **인터페이스 우선**: Repository는 인터페이스로, 구현체는 `Impl` 접미사
- **미확인 항목**: `// TODO: 확인 필요` 주석 필수

## 산출물 포맷

파일명: `클래스설계서_{사업명}_{YYYYMMDD}.md` (`${CLAUDE_SKILL_DIR}/template.md` 사용)
