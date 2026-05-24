---
name: ai-dlc-change-complete
description: AI-DLC 변경 관리 스킬. CR-NNN을 받아 change-log.md 완료 처리, SKILL.md 변경 이력 추가, 완료 요약 출력을 자동화한다. "CR 완료 처리", "변경 완료", "CR-NNN 닫아줘", "변경 이력 업데이트", "CR 완료 등록" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC 변경 완료 처리

> CR 변경이 모두 완료된 후 실행하는 최종 단계.  
> change-log.md 상태를 ✅ 완료로 갱신하고, SKILL.md 이력에 기록한 후 완료 요약을 출력한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "CR 완료 처리", "변경 완료", "CR-NNN 닫아줘"
- "변경 이력 업데이트", "CR 완료 등록"
- 모든 revise + validate + 코드 반영이 끝난 후 마무리 요청

## 입력

### 필수
- CR-NNN 번호 (예: CR-004)

### 선택
- 변경된 파일 목록 (없으면 change-log.md 기존 내용 활용)
- 영향받은 산출물과 버전 (예: "화면정의서 v1.2")
- git 커밋 해시 (없으면 "(미확정)"으로 표시)

## 처리 절차

1. **change-log.md 읽기**
   - `docs/change-log.md` 탐색
   - 없으면 "docs/change-log.md 를 찾을 수 없습니다. 먼저 /ai-dlc-change-register 를 실행하세요." 출력 후 중단

2. **CR-NNN 존재 확인**
   - CR 목록 테이블에서 지정된 CR-NNN 행 탐색
   - 없으면 "CR-NNN 항목이 존재하지 않습니다. CR 번호를 확인하세요." 출력 후 중단

3. **CR 목록 테이블 행 업데이트**
   현재 상태(`⏳ 대기` 또는 `🔄 진행`) → `✅ 완료`로 변경:
   ```markdown
   | CR-NNN | ... | ✅ 완료 | YYYY-MM-DD | YYYY-MM-DD |
   ```
   - 완료일 컬럼을 오늘 날짜로 채움

4. **CR 상세 섹션 완료 내용 추가**
   해당 `### CR-NNN` 섹션 내에 완료 블록 추가:
   ```markdown
   **완료일**: YYYY-MM-DD  
   **완료 상태**: ✅ 완료

   **실행한 스킬**
   - /ai-dlc-xxx-revise
   - /ai-dlc-xxx-validate
   - /ai-dlc-consistency-check
   - (코드 반영)

   **변경 파일**
   - 파일 경로 1
   - 파일 경로 2

   **관련 산출물 버전**
   - 설계산출물명: vX.X

   **git 커밋**: (해시 또는 미확정)
   ```
   - 입력된 정보가 없으면 "(미확정)" 또는 "(상세 섹션 참조)"로 기록

5. **SKILL.md 변경 이력 섹션 갱신**
   - `Harness/SKILL.md` 또는 프로젝트 루트 `SKILL.md` 탐색
   - 파일 내 `## 변경 이력` 섹션 찾기 (없으면 파일 끝에 섹션 생성)
   - 행 추가:
   ```markdown
   | CR-NNN | YYYY-MM-DD | CR-유형 | 변경 제목 요약 | 실행 스킬 | ✅ 완료 |
   ```
   - SKILL.md가 없거나 경로 불명확하면 "SKILL.md를 찾을 수 없어 생략합니다. 수동으로 추가하세요." 경고 출력

6. **완료 요약 터미널 출력**

## 산출물

- `docs/change-log.md` 업데이트 (Edit 사용)
- `Harness/SKILL.md` 또는 프로젝트 `SKILL.md` 업데이트 (Edit 사용, 파일 존재 시)
- 완료 요약 보고서 터미널 출력

## 완료 요약 출력 포맷

```markdown
## CR 완료 처리 결과

**CR-NNN** — [변경 제목]  
**완료일**: YYYY-MM-DD  
**유형**: CR-TYPE

---

### 처리 내용
- change-log.md: ✅ 완료 처리 (완료일 기록)
- SKILL.md: ✅ 이력 추가 (또는 "⚠️ 수동 추가 필요")

### 변경 요약
[CR 상세 섹션의 변경 원인 2~3줄 요약]

---

### 다음 단계 (선택)
- 다른 대기 중인 CR이 있으면: /ai-dlc-change-register [다음 변경 내용]
- 전체 일관성 재확인: /ai-dlc-consistency-check @설계산출물/
```

## 엣지 케이스

- **CR이 이미 ✅ 완료**: "CR-NNN은 이미 완료 처리되었습니다. (완료일: YYYY-MM-DD)" 출력 후 중단
- **change-log.md 없음**: "docs/change-log.md를 찾을 수 없습니다. /ai-dlc-change-register를 먼저 실행하세요." 안내
- **SKILL.md 없음 또는 다중 SKILL.md**: 경고 출력 + 발견된 경로 목록 제시, 사용자가 선택하도록 안내
- **완료 정보 미제공 (파일 목록, 커밋 등)**: "(미확정)"으로 기록하고 "입력 정보가 부족하여 일부 항목을 미확정으로 처리했습니다. 이후 직접 보완하세요." 안내
- **여러 CR 동시 완료 요청**: CR 목록을 받아 순서대로 처리 (CR-001, CR-002, ...) — 각 CR마다 확인 후 진행
