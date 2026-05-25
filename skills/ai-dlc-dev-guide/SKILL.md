---
name: ai-dlc-dev-guide
description: AI-DLC 납품/인도 스킬. API설계서·데이터설계서·클래스설계서·요구사항정의서·프로젝트 설정 파일을 분석하여 개발 환경 설정부터 API 참조·DB 스키마·배포 방법까지 담은 개발자 가이드를 생성한다. "개발자 가이드 만들어줘", "개발 가이드 작성해줘", "기술 문서 만들어줘", "API 사용 가이드", "유지보수 가이드", "인수인계 기술 문서" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write
---

# AI-DLC 개발자 가이드

> API설계서·데이터설계서·클래스설계서·프로젝트 설정 파일을 읽어
> 개발 환경 설정·코드 구조·API 참조·DB 스키마·배포 방법을 담은
> `개발자가이드_*.md`를 생성한다.
> 새로운 개발자가 이 문서만으로 시스템을 이해하고 개발·유지보수할 수 있도록 구성한다.

## 트리거

- "개발자 가이드 만들어줘", "개발 가이드 작성해줘"
- "기술 문서 만들어줘", "API 사용 가이드 만들어줘"
- "유지보수 가이드 작성해줘", "인수인계 기술 문서 만들어줘"
- "개발 환경 설정 문서 만들어줘", "기술 인수인계서"

## 입력

### 자동 탐색 (설계 산출물)
- `openapi_*.yaml` 또는 `API설계서_*.md` — API 명세
- `데이터설계서_*.md` — DB 스키마
- `클래스설계서_*.md` — 코드 구조
- `요구사항정의서_*.md` — 기술스택

### 자동 탐색 (프로젝트 설정)
- `pom.xml` 또는 `build.gradle` (Spring Boot)
- `package.json` (React/Vue/Next.js)
- `application.yml` 또는 `application.properties`
- `.env.example`
- `Dockerfile` 또는 `docker-compose.yml`

### 선택
- `비즈니스규칙_*.md` — 핵심 비즈니스 규칙 요약
- `프로그램분석서_*.md` — 소스코드 구조

## 처리 절차

1. **설계 산출물 탐색 및 읽기**
   - Glob → Read (있는 파일만)

2. **프로젝트 설정 파일 탐색**
   - 루트 및 하위 디렉터리에서 설정 파일 탐색
   - 기술 스택 타입 판별: Spring Boot vs Node.js vs 둘 다

3. **기술스택 및 Prerequisites 추출**
   - 요구사항정의서 기술스택 표 또는 설정 파일에서 추출
   - 필요 도구 목록 (JDK, Node.js, DB 등) + 권장 버전

4. **코드 구조 정리**
   - 클래스설계서 또는 패키지 구조에서 디렉터리 트리 생성
   - 각 디렉터리/패키지 역할 설명

5. **API 참조 표 생성**
   - YAML: paths 섹션 파싱 → Method·Endpoint·설명·인증·요청Body·응답 표
   - Markdown: operationId 기준 표 추출

6. **DB 스키마 요약**
   - 데이터설계서의 테이블별 핵심 컬럼 요약 표

7. **환경 설정 단계 작성**
   - 설정 파일 기반 단계별 명령어 제공

8. **배포 방법 작성**
   - Dockerfile 존재 → Docker 배포 명령어
   - package.json scripts → npm run build/start
   - pom.xml → mvn package + java -jar 실행
   - .env.example → 환경 변수 목록

9. **template.md 기반 파일 생성**

## 산출물

파일명: `개발자가이드_{사업명}_{YYYYMMDD}.md`
- 출력 시 절대 경로 + 포함된 API 수 + 테이블 수

## 엣지 케이스

- **설계 산출물 없이 소스코드만 있는 경우**: `ai-dlc-program-spec` 먼저 실행 권장 안내
- **Spring Boot + React 풀스택**: 백엔드·프론트엔드 각 섹션으로 분리
- **API 설계서가 YAML 형식**: `paths` 키 파싱하여 엔드포인트 표 자동 생성
- **환경 변수 민감 정보**: 실제 값 대신 `${변수명}` 형태로만 표기
