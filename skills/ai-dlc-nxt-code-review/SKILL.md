---
name: ai-dlc-nxt-code-review
description: AI-DLC 개발단계(프론트엔드-Next.js) 스킬. Next.js App Router 코드 품질을 검토한다. "Next.js 코드 검토해줘", "App Router 코드 리뷰", "RSC 코드 검토", "Next.js 코드 리뷰", "NX 코드 품질 검사" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC Next.js App Router 코드 품질 검토

Next.js 15 App Router 기반 소스코드를 검토하여 RSC/CC 경계 오류, 보안 취약점, 성능 문제를 발견하고 코드품질검토 보고서를 생성한다.

## 트리거

- "Next.js 코드 검토해줘", "App Router 코드 리뷰"
- "RSC 코드 검토", "Next.js 코드 리뷰"
- "NX 코드 품질 검사", "Next.js 품질 검토"

---

## 입력

### 필수
- 검토 대상 디렉터리 또는 파일 목록

### 선택
- 구현 계획서 (`Next.js구현계획_*.md`) — RSC/CC 분류 기준 확인용

---

## 검토 절차

1. `Glob`으로 `app/`, `components/`, `actions/` 파일 목록 수집
2. 각 파일을 `Read`하여 NX 이슈 코드 기준으로 검토
3. `ai-dlc-fe-code-review`의 TC/LC/PF/SC/A11Y 기준도 함께 적용
4. 이슈 목록 작성 (파일명:줄번호, 이슈코드, 설명, 수정 방향)
5. 심각도별 요약 + 전체 점수 산출

---

## NX 이슈 코드 (Next.js 고유)

| 코드 | 검사 항목 | 심각도 |
|:---|:---|:---:|
| NX-001 | `'use client'` 과다 사용 — RSC로 전환 가능 | 높음 |
| NX-002 | CC에서 직접 DB·서버 로직(`prisma`, `fs`) 호출 | 높음 |
| NX-003 | `<img>` 태그 사용 — `next/image` 미사용 | 중간 |
| NX-004 | `<a href>` 태그 사용 — `next/link` 미사용 | 중간 |
| NX-005 | Route Handler에 인증 체크 누락 | 높음 |
| NX-006 | Server Action에 Zod 유효성 검사 누락 | 높음 |
| NX-007 | `fetch()` 캐시 설정 누락 (캐싱 전략 미정의) | 중간 |
| NX-008 | `NEXT_PUBLIC_` 없는 env var를 CC에서 접근 | 높음 |
| NX-009 | Server Action 후 `revalidatePath`/`revalidateTag` 누락 | 중간 |
| NX-010 | `error.tsx` 또는 `loading.tsx` 미정의 | 낮음 |

## 공통 이슈 코드 (fe-* 재사용)

| 코드 | 검사 항목 | 심각도 |
|:---|:---|:---:|
| TC-001 | `any` 타입 사용 | 높음 |
| TC-002 | 반환 타입 미선언 | 중간 |
| TC-003 | null/undefined 안전 처리 누락 | 높음 |
| SC-001 | 하드코딩된 비밀값 (토큰·패스워드) | 매우 높음 |
| PF-001 | 불필요한 `'use client'` — RSC로 대체 가능 | 중간 |
| A11Y-001 | `alt` 속성 누락 (`next/image`) | 중간 |
| A11Y-002 | `aria-label` 누락 (아이콘 버튼) | 중간 |

---

## 산출물 형식

```markdown
# Next.js 코드품질검토 보고서

| 항목 | 내용 |
|:---|:---|
| 작성일 | YYYY-MM-DD |
| 검토 범위 | app/, components/, actions/ |
| 총 이슈 수 | N건 (높음 X / 중간 Y / 낮음 Z) |

## 이슈 목록

| # | 파일:줄 | 코드 | 설명 | 수정 방향 |
|:---:|:---|:---:|:---|:---|
| 1 | components/UserForm.tsx:12 | NX-001 | `'use client'` 불필요 — 이벤트 핸들러 없음 | RSC로 전환 |
...

## 심각도별 요약

- 매우 높음: N건 → 즉시 수정 필요
- 높음: N건 → 이번 스프린트 내 수정
- 중간: N건 → 다음 스프린트 수정 권장
- 낮음: N건 → 여유 시 수정
```

---

## 산출물

- `코드품질검토_{YYYYMMDD}.md` — 이슈 목록 + 수정 방향
