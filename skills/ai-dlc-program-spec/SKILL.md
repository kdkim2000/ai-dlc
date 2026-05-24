---
name: ai-dlc-program-spec
description: AI-DLC 코드분석단계 스킬. 소스코드를 분석해 표준 프로그램 분석서를 생성한다. "프로그램 분석서", "코드 분석서 만들어줘", "소스코드 문서화", "프로그램 명세서", "코드 설명서", "코드 분석해줘", "소스 분석서" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC 프로그램 분석서 생성

소스코드를 읽어 시스템 구조, 주요 컴포넌트, 처리 흐름, 외부 연계를 파악하고 **표준 프로그램 분석서**를 작성한다. 개발자가 코드를 보지 않고도 시스템을 이해할 수 있는 수준으로 문서화한다.

공통 출력 정책: `${CLAUDE_SKILL_DIR}/../ai-dlc-common/references/output-policy.md` 참조.

## 트리거

- "프로그램 분석서", "코드 분석서 만들어줘", "소스코드 문서화", "프로그램 명세서"
- "코드 설명서", "코드 분석해줘", "소스 분석서", "시스템 분석서"
- 소스코드 경로를 주며 "이 코드 분석해줘"라고 할 때

## 입력

- **필수**: 분석 대상 경로 (디렉터리 또는 특정 파일)
- **선택**: 시스템명, 분석 깊이 (전체 / 핵심만), 특정 기능 집중 분석

## 분석 절차

1. **프로젝트 구조 파악**: `Glob`으로 전체 파일 트리 수집, 디렉터리 구조 도식화
2. **언어/프레임워크 감지**: 확장자 + 설정 파일(package.json, pom.xml, setup.py 등) 기반 자동 감지
3. **진입점 분석**: `main`, `index`, `app`, `server`, `Application.java` 등 시작점 탐지 후 `Read`
4. **컴포넌트 목록 추출**: 클래스, 함수, 라우터, 미들웨어, 서비스, 리포지토리 `Grep`으로 추출
5. **외부 연계 탐지**:
   - DB: `mongoose`, `prisma`, `JPA`, `SQLAlchemy`, `pg.Pool`, `mysql.createConnection` 등
   - HTTP: `fetch`, `axios`, `HttpClient`, `RestTemplate`, `requests.get` 등
   - 파일 I/O: `fs.readFile`, `open(`, `Files.read` 등
6. **보안 경고 탐지**: 하드코딩 시크릿 패턴 (`password=`, `api_key=`, `secret=` 등)
7. **`template.md` 기반 분석서 작성**

## 지원 언어/프레임워크 특화 탐지

| 언어 | 프레임워크 | 특화 탐지 패턴 |
|:---|:---|:---|
| TypeScript | Next.js | `pages/`, `app/`, `getServerSideProps`, `API Routes` |
| TypeScript | NestJS | `@Controller`, `@Injectable`, `@Module` |
| TypeScript | Express | `router.get/post/put/delete`, `app.use` |
| Python | FastAPI | `@app.get`, `@router.post`, `Depends(` |
| Python | Django | `urlpatterns`, `views.py`, `models.py` |
| Java | Spring Boot | `@RestController`, `@Service`, `@Repository` |
| Go | 표준 | `http.HandleFunc`, `gin.Default()` |

## 산출물 포맷

파일명: `프로그램분석서_{시스템명}_{YYYYMMDD}.md`

`${CLAUDE_SKILL_DIR}/template.md` 골격 사용:

1. **시스템 개요** (언어, 프레임워크, 분석 범위, LOC 대략)
2. **디렉터리/모듈 구조** (트리 + 각 디렉터리 역할)
3. **주요 컴포넌트 목록** `| 컴포넌트명 | 유형 | 역할 | 주요 함수/메서드 |`
4. **처리 흐름** (주요 유스케이스별 시퀀스 텍스트 다이어그램)
5. **외부 연계** `| 연계 대상 | 유형(DB/HTTP/파일) | 연결 방식 | 비고 |`
6. **특이 사항** (기술 부채, TODO/FIXME 현황, 보안 경고)

## 작성 원칙

- 컴포넌트 수가 50개 이상이면 핵심 20개를 상세 기술, 나머지는 목록만 표시
- 처리 흐름은 상위 3~5개 주요 유스케이스에 집중 (예: 로그인, 주요 CRUD)
- `node_modules`, `dist`, `build`, `.git`, `__pycache__` 제외
- 보안 경고 발견 시 "6. 특이 사항"에 별도 섹션으로 강조 표시

## 엣지 케이스

- **단일 파일 프로젝트**: 함수 단위로 분석서 작성
- **다중 언어 혼재**: 언어별 섹션 분리
- **레거시 코드 (주석 없음)**: 패턴 기반 역추론 후 `<!-- TODO: 추정값, 확인 필요 -->` 표시
- **파일 수 100개 초과**: 핵심 파일 자동 우선순위화 + "전체 파일 목록은 별첨" 안내
