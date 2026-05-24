---
name: ai-dlc-nxt-impl-plan
description: AI-DLC 개발단계(프론트엔드-Next.js) 스킬. Next.js App Router 구현 전략 계획 문서를 생성한다. "Next.js 구현 계획 세워줘", "App Router 구현 전략", "Next.js 화면 구현 순서", "RSC 설계 계획", "Next.js 개발 계획서" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob Write Edit
---

# AI-DLC Next.js App Router 구현 전략 계획

화면설계서(SCR-NNN)·API설계서·유즈케이스(UC-NNN)를 입력받아 Next.js App Router 기반 구현 전략 계획서를 생성한다.

## 트리거

- "Next.js 구현 계획 세워줘", "App Router 구현 전략"
- "Next.js 화면 구현 순서", "RSC 설계 계획"
- "Next.js 개발 계획서", "App Router 설계 전략"

---

## 입력

### 필수
- 화면설계서 (SCR-NNN 목록)
- API 설계서 (operationId 또는 Route Handler 목록)

### 선택
- 유즈케이스 문서 (UC-NNN)
- `ai-dlc-nxt-project-setup` 산출물 (기술스택 확인용)

---

## 계획 수립 절차

1. **화면 목록 정리**: SCR-NNN → `app/` 라우트 경로 매핑
2. **RSC/CC 경계 결정**:
   - RSC(기본): 데이터 조회, 정적 UI, SEO 필요 페이지
   - CC(`'use client'`): useState/useEffect 사용, 이벤트 핸들러, 브라우저 API
3. **라우트 구조 설계**: Route Groups(`(auth)`, `(dashboard)`) + Dynamic Routes(`[id]`) + Parallel Routes(`@modal`)
4. **데이터 패칭 전략**:
   - RSC: `fetch()` with `{ cache: 'force-cache' | 'no-store', next: { revalidate: N } }`
   - CC: TanStack Query (`useQuery`, `useMutation`)
   - 폼 뮤테이션: Server Actions 우선
5. **Server Actions vs Route Handlers 선택**:
   - Server Actions: 폼 제출, 내부 뮤테이션 (브라우저 전용)
   - Route Handlers: 외부 API 노출, 파일 업로드, 웹훅, 모바일 앱 연동

---

## RSC / CC 판단 기준

| 조건 | 권장 |
|:---|:---|
| 데이터베이스 직접 조회 | RSC |
| SEO가 중요한 페이지 | RSC |
| 정적 텍스트·목록 표시 | RSC |
| useState, useReducer 사용 | CC |
| onClick, onChange 등 이벤트 핸들러 | CC |
| useEffect, useRef 사용 | CC |
| localStorage, window 접근 | CC |
| TanStack Query useQuery/useMutation | CC |
| Toast, Dialog 등 UI 상태 | CC |

---

## 산출물

- `Next.js구현계획_{YYYYMMDD}.md` — 라우트 구조 + RSC/CC 분류 + 데이터 패칭 전략 포함
