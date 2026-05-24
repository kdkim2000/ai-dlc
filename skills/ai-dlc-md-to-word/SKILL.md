---
name: ai-dlc-md-to-word
description: AI-DLC 코드분석단계 스킬. 마크다운(.md) 파일을 워드(.docx) 문서로 변환한다. pandoc을 우선 사용하고, 없으면 대안을 안내한다. "워드로 변환", "docx로 바꿔줘", "마크다운을 워드로", "Word 문서 만들어줘", "md to docx", "워드 파일로", "문서 변환" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Bash(pandoc *) Bash(python *) Bash(pip *) Bash(where *) Bash(which *) Read
---

# AI-DLC 마크다운 → 워드 문서 변환

마크다운(`.md`) 파일을 Microsoft Word(`.docx`) 형식으로 변환한다. **pandoc**을 우선 사용하며, 미설치 시 Python(`python-docx`)으로 대체하거나 설치 안내를 제공한다.

## 트리거

- "워드로 변환", "docx로 바꿔줘", "마크다운을 워드로", "Word 문서 만들어줘"
- "md to docx", "워드 파일로", "문서 변환", ".docx로 저장해줘"
- 마크다운 파일 경로를 주며 "이걸 워드로 바꿔줘"라고 할 때

## 입력

- **필수**: 변환할 마크다운 파일 경로 (또는 현재 대화에서 생성된 마크다운 내용)
- **선택**:
  - 출력 파일 경로/이름 (기본: 입력 파일명에서 확장자만 `.docx`로 변경)
  - reference.docx 경로 (AI-DLC 표준 스타일 적용용 — 없으면 pandoc 기본 스타일)

## 변환 절차

### Step 1: 도구 확인

```bash
# pandoc 확인 (Windows)
where pandoc
# 또는 pandoc --version
```

### Step 2-A: pandoc 사용 (권장)

```bash
# 기본 변환
pandoc {input}.md -o {output}.docx

# 스타일 적용 (reference.docx 있을 때)
pandoc {input}.md -o {output}.docx --reference-doc={reference}.docx

# 수식·표 포함 시
pandoc {input}.md -o {output}.docx --reference-doc={reference}.docx --standalone
```

### Step 2-B: Python 대체 (pandoc 없을 때)

python-docx 라이브러리 사용:
```bash
pip install python-docx
python -c "
from docx import Document
# 마크다운 기본 파싱 후 docx 생성
# 제한: 복잡한 표/코드블록은 서식 손실 가능
"
```

### Step 2-C: 수동 안내 (둘 다 없을 때)

설치 안내:
- **pandoc 설치**: https://pandoc.org/installing.html (Windows: `winget install pandoc` 또는 MSI 다운로드)
- **python-docx**: `pip install python-docx`

## 변환 품질 기준

| 요소 | pandoc | python-docx | 비고 |
|:---|:---:|:---:|:---|
| 제목/본문 서식 | 완전 | 부분 | |
| 표 | 완전 | 부분 | |
| 코드 블록 | 완전 | 제한 | |
| 이미지 | 완전 | 제한 | 로컬 경로만 |
| 수식(LaTeX) | 완전 | 미지원 | |
| 페이지 번호 | reference.docx 기준 | 미지원 | |

## reference.docx 활용

AI-DLC 표준 문서 스타일을 적용하려면 `reference.docx`를 준비한다:
1. 빈 Word 문서에서 제목1/제목2/본문/표 스타일을 정의
2. 파일을 `reference.docx`로 저장
3. 변환 시 `--reference-doc=reference.docx` 옵션 추가

## 출력

- **파일 생성**: `{원본파일명}.docx` (또는 지정 경로)
- **완료 보고**: 파일 절대 경로 + 크기 출력
- **경고**: python-docx 사용 시 서식 손실 가능 항목 안내

## 엣지 케이스

- **대화창의 마크다운 내용 변환 요청**: 먼저 임시 `.md` 파일로 저장 후 변환
- **이미지가 포함된 md**: 로컬 이미지만 포함 가능, 원격 URL 이미지는 미포함 경고
- **한글 포함**: pandoc은 한글 완벽 지원; python-docx는 UTF-8 인코딩 명시 필요
- **파일이 이미 존재**: 덮어쓰기 전 확인
