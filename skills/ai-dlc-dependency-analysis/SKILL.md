---
name: ai-dlc-dependency-analysis
description: AI-DLC 코드분석단계 스킬. 모듈 간 의존관계와 외부 패키지 의존성을 분석하고 순환 의존을 탐지한다. "의존성 분석", "모듈 의존관계", "패키지 분석", "순환 의존", "dependency analysis", "의존성 그래프", "모듈 의존도 파악", "어떤 패키지 쓰는지" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC 코드 의존성 분석

소스코드의 외부 패키지 의존성과 내부 모듈 의존 관계를 분석하고 순환 의존성·위험 의존성을 탐지하여 **의존성 분석서**를 산출한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "의존성 분석", "모듈 의존관계", "패키지 분석", "순환 의존"
- "dependency analysis", "의존성 그래프", "모듈 의존도 파악"
- "어떤 패키지 쓰는지", "라이브러리 현황 파악", "의존성 파악해줘"
- 특정 파일/모듈을 언급하며 "의존 관계 그려줘"라고 할 때

## 입력

- **필수**: 분석 대상 경로 (디렉터리)
- **선택**: 제외 경로 (기본: `node_modules`, `.git`, `dist`, `build`, `__pycache__`)

## 분석 절차

1. **패키지 파일 탐지**: `Glob`으로 아래 파일 탐지 후 `Read`
   - Node.js: `package.json`, `package-lock.json`, `yarn.lock`
   - Python: `requirements.txt`, `pyproject.toml`, `setup.py`, `Pipfile`
   - Java/Kotlin: `pom.xml`, `build.gradle`, `build.gradle.kts`
   - Go: `go.mod`, `go.sum`
2. **외부 패키지 목록 추출**: 패키지 파일 파싱 → 패키지명·버전·용도 목록 생성
3. **내부 모듈 import 추출**: `Grep`으로 각 언어별 import/require 구문 전체 탐색
   - TypeScript/JS: `import.*from`, `require(`
   - Python: `^from\s`, `^import\s`
   - Java: `^import\s`
   - Go: `"import"` 블록
4. **의존 그래프 구성**: 파일별 의존 관계 매핑 → 텍스트 다이어그램 생성
5. **순환 의존성 탐지**: 깊이 우선 탐색(DFS) 방식으로 순환 경로 탐지
6. **위험 의존성 식별**: deprecated 마커, 매우 오래된 버전(2년 이상), 알려진 취약 패키지 패턴

## 산출물 포맷

파일명: `의존성분석_{시스템명}_{YYYYMMDD}.md`

`${CLAUDE_SKILL_DIR}/template.md` 골격 사용:

1. **외부 패키지 의존성** (패키지명·버전·용도·위험 여부)
2. **내부 모듈 의존 관계** (텍스트 계층 다이어그램)
3. **순환 의존성 목록**
4. **위험 의존성** (deprecated, 오래된 버전, 취약 패턴)
5. **개선 권고**

## 언어별 import 탐색 패턴

| 언어 | 외부 패키지 파일 | import 패턴 |
|:---|:---|:---|
| TypeScript/JS | `package.json` | `import.*from ['"]` |
| Python | `requirements.txt`, `pyproject.toml` | `^from\s+\w`, `^import\s+\w` |
| Java | `pom.xml`, `build.gradle` | `^import\s+[a-z]` |
| Go | `go.mod` | `"` 로 시작하는 import 경로 |

## 작성 원칙

- `node_modules`, `.git`, `dist`, `build`, `__pycache__` 제외
- 내부 모듈 의존 그래프는 레벨 3 이하까지만 표시 (더 깊으면 요약)
- 순환 의존 발견 시 위험도 `높음` 자동 부여
- 정적 분석 한계(동적 import, 런타임 의존성)는 명기

## 엣지 케이스

- **패키지 파일 없음**: 소스 내 import 구문만으로 추정 분석 + 경고 표시
- **다중 언어 혼재**: 언어별 섹션 분리
- **모노레포**: 패키지별로 의존성 분리 분석
