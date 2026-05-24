# AI-DLC 분석단계 산출물 품질 기준

validate 스킬이 검증 시 참조하는 품질 기준 체크리스트.

## 공통 품질 기준 (전체 산출물)

- [ ] 모든 필수 컬럼이 채워져 있다
- [ ] ID 채번이 연속적이고 중복이 없다
- [ ] 추측 내용(`<!-- TODO -->`)이 5개 이하다 (초안 기준)
- [ ] 문서 버전 이력 표가 있다
- [ ] 한국어가 일관되게 사용되었다

## 비즈니스 규칙 (BR) 품질 기준

### 완전성
- [ ] 모든 BR에 `조건(When/If)`이 명시되어 있다
- [ ] 모든 BR에 `행동(Then)`이 명시되어 있다
- [ ] 우선순위가 설정되어 있다 (높음/중간/낮음)
- [ ] 상태가 설정되어 있다 (초안/검증중/확정/폐기)

### 일관성
- [ ] 동일 조건에 상반된 행동을 가진 규칙 쌍이 없다 (모순)
- [ ] 실질적으로 동일한 내용의 규칙이 중복 등록되지 않았다
- [ ] 도메인 분류가 일관적이다

### 명확성
- [ ] 조건이 "~인 경우", "~이면" 등 명확한 트리거로 표현되었다
- [ ] 행동이 "~한다", "~처리한다" 등 구체적 동작으로 표현되었다
- [ ] 해석이 두 가지 이상으로 가능한 표현이 없다

### 검증 이슈 유형
| 유형 | 설명 | 심각도 |
|:---|:---|:---:|
| 모순 | 동일 조건, 상반 행동 | 높음 |
| 중복 | 실질 동일 규칙 복수 등록 | 중간 |
| 모호 | 다의적 해석 가능 표현 | 중간 |
| 누락 | 중요 예외 케이스 미처리 | 중간 |
| 형식오류 | ID 중복, 필수값 누락 | 낮음 |

## 도메인 표준 용어 (TM) 품질 기준

### 완전성
- [ ] 모든 용어에 정의가 작성되어 있다
- [ ] 도메인이 분류되어 있다
- [ ] 상태가 설정되어 있다

### 일관성
- [ ] 같은 개념을 다른 용어로 각각 정의한 쌍이 없다 (동의어 충돌)
- [ ] 정의가 다른 용어를 사용해 순환 정의되지 않았다
- [ ] 동의어/유사어 컬럼이 용어사전 내 다른 행과 일치한다

### 명확성
- [ ] 정의가 최소 1개 완전한 문장이다 (단어 나열 불가)
- [ ] 사용 예시가 제공되어 있다
- [ ] 출처(내부정의/ISO/국가표준 등)가 명시되어 있다

### 검증 이슈 유형
| 유형 | 설명 | 심각도 |
|:---|:---|:---:|
| 정의 모호성 | 순환 정의 또는 지나치게 짧은 정의 | 높음 |
| 동의어 충돌 | 같은 개념의 두 용어가 별개로 등록 | 높음 |
| 도메인 불일치 | 잘못된 도메인 배정 | 중간 |
| 미사용 용어 | 요구사항 어디에도 등장 안 함 | 낮음 |
| 형식오류 | 정의 누락, 상태값 오류 | 낮음 |

## 서비스 카탈로그 품질 기준

- [ ] 모든 FR 항목이 최소 하나의 서비스에 배정되었다
- [ ] 서비스명이 비즈니스 도메인을 명확히 표현한다
- [ ] 서비스 간 경계가 명확하다 (하나의 FR이 두 서비스에 중복 배정된 경우 의도적임을 표시)
- [ ] 카테고리 분류가 일관적이다

## 기능/비기능 요구사항 품질 기준

### 기능 요구사항
- [ ] 액터가 명시되어 있다
- [ ] 사전조건과 사후조건이 있다 (간단한 기능도 최소 1개)
- [ ] 우선순위가 설정되어 있다

### 비기능 요구사항
- [ ] 목표값에 측정 단위가 명시되어 있다 (예: "빠르게" → "응답시간 1초 이내")
- [ ] 검증방법이 구체적이다 (부하 테스트 도구명, 보안 점검 방법 등)
- [ ] 분류(PR/SR/QR 등)가 `ai-dlc-requirements/references/id-codes.md` 기준에 맞다

---

## 유즈케이스 (UC) 품질 기준

### 완전성
- [ ] 모든 UC에 기본 흐름이 있다
- [ ] 모든 UC에 최소 1개 대안 흐름 또는 예외 흐름이 있다
- [ ] 연계 FR-NNN이 명시되어 있다
- [ ] 액터가 명시되어 있다

### 일관성
- [ ] FR 커버리지 ≥ 100% (모든 FR이 최소 1개 UC와 연계)
- [ ] 동일 시나리오가 다른 UC-ID로 중복 등록되지 않았다
- [ ] 액터 권한과 UC 접근 제약이 일치한다

### 검증 이슈 유형
| 유형 | 설명 | 심각도 |
|:---|:---|:---:|
| 누락 | FR-UC 미연계 | 높음 |
| 불완전 | 예외 흐름 없음 | 중간 |
| 모순 | 액터 권한 불일치 | 높음 |
| 중복 | 실질 동일 UC 복수 등록 | 중간 |
| 형식오류 | UC-ID 중복, 필수 섹션 누락 | 낮음 |

---

## 화면 설계 (SCR) 품질 기준

### 완전성
- [ ] 모든 UC 기본 흐름이 최소 1개 화면으로 표현된다
- [ ] UC-화면 커버리지 ≥ 90%
- [ ] 모든 FORM 화면에 취소/이탈 경로가 있다

### 일관성
- [ ] 화면 I/O 필드가 연계 API 스키마와 일치한다
- [ ] 역할별 접근 정의와 화면 조건부 표시 규칙이 일치한다

### UX 기준
- [ ] 필수 입력 필드에 유효성 규칙이 있다
- [ ] FORM 화면에 오류 메시지 처리 이벤트가 있다

### 검증 이슈 유형
| 유형 | 설명 | 심각도 |
|:---|:---|:---:|
| 누락 | UC 흐름이 화면으로 미표현 | 높음 |
| 불일치 | 화면 I/O ≠ API 파라미터 | 높음 |
| 권한 충돌 | 역할별 접근 정의 vs. 화면 표시 조건 불일치 | 중간 |
| UX 결함 | 유효성 규칙 없음, 오류 메시지 없음, 이탈 경로 없음 | 중간 |
| 형식오류 | SCR-ID 중복, 필수 섹션 누락 | 낮음 |

---

## API 설계 품질 기준

### RESTful 준수
- [ ] 경로가 명사형 리소스다 (동사형 금지: `/getUser` 등)
- [ ] HTTP 메서드가 의미론에 맞다 (GET=조회, POST=생성, PUT/PATCH=수정, DELETE=삭제)
- [ ] 컬렉션과 단건이 구분된다 (`/users` vs `/users/{id}`)

### 완전성
- [ ] 모든 엔드포인트에 `summary`가 있다
- [ ] 모든 엔드포인트에 responses 200/400/401이 정의되어 있다
- [ ] 모든 스키마 `$ref` 참조 대상이 존재한다
- [ ] UC-API 커버리지 ≥ 90%

### 보안
- [ ] 공개 엔드포인트(로그인, 헬스체크 제외) 모두에 `security` 적용
- [ ] `securitySchemes`가 정의되어 있다

### 검증 이슈 유형
| 유형 | 설명 | 심각도 |
|:---|:---|:---:|
| UC 커버리지 누락 | UC 흐름 단계가 API로 미표현 | 높음 |
| RESTful 위반 | 동사형 경로, 잘못된 HTTP 메서드 | 높음 |
| 스키마 불완전 | 스키마 미정의, TODO 미해결 | 중간 |
| 보안 누락 | security 미적용 엔드포인트 | 중간 |
| 형식오류 | operationId 중복, summary 누락 | 낮음 |

---

## 데이터 설계 품질 기준

### 정규화
- [ ] 1NF 준수: 반복 그룹 컬럼 없음
- [ ] 2NF 준수: 복합 PK 테이블에 부분 종속 없음
- [ ] 3NF 준수: 이행 종속 없음

### 완전성
- [ ] 모든 테이블에 PK가 있다
- [ ] 공통 컬럼(`created_at`, `updated_at`, `created_by`)이 모든 테이블에 있다
- [ ] 모든 FK 참조 테이블이 존재한다
- [ ] FK 컬럼에 인덱스가 있다

### 일관성
- [ ] API 설계서 스키마 필드와 테이블 컬럼이 일치한다
- [ ] 테이블명은 `TB_대문자_스네이크케이스`다
- [ ] 컬럼명은 `소문자_스네이크케이스`다

### 검증 이슈 유형
| 유형 | 설명 | 심각도 |
|:---|:---|:---:|
| 정규화 위반 | 1/2/3NF 위반 | 높음 |
| 참조 무결성 | FK 참조 대상 없음, ON DELETE 정책 미정의 | 높음 |
| API 스키마 불일치 | API 필드 ≠ 컬럼 | 중간 |
| 누락 | PK 없는 테이블, 공통 컬럼 누락 | 중간 |
| 명명 규칙 위반 | 테이블/컬럼명 규칙 위반 | 낮음 |

---

## 클래스 설계 품질 기준

### SOLID 원칙
- [ ] SRP: 단일 클래스 메서드 수 ≤ 10개 (휴리스틱)
- [ ] 순환 의존이 없다
- [ ] 레이어 의존 방향이 Controller → Service → Repository → Domain이다

### 완전성
- [ ] 모든 클래스에 레이어가 지정되어 있다
- [ ] 모든 메서드에 파라미터·반환 타입이 정의되어 있다
- [ ] UC-클래스 커버리지 ≥ 90%

### 검증 이슈 유형
| 유형 | 설명 | 심각도 |
|:---|:---|:---:|
| 순환 의존 | A→B→C→A 형태 | 높음 |
| 레이어 경계 위반 | Controller→Repository 직접 참조 등 | 높음 |
| SRP 위반 | 메서드 수 > 10개 | 중간 |
| UC 커버리지 누락 | UC 흐름 단계가 메서드로 미표현 | 중간 |
| 형식오류 | CLS-ID 중복, 타입 미정의 | 낮음 |

---

## 개발단계(백엔드-Spring Boot) 품질 기준 (PLAN-005)

### 레이어 컨벤션 (LC)

- [ ] Controller는 Service만 의존한다 (Mapper·Repository 직접 참조 금지)
- [ ] Service는 Mapper/Repository만 의존한다 (타 Service 직접 호출 지양)
- [ ] `@Transactional`은 Service 레이어에만 선언한다 (Controller 선언 금지)
- [ ] `@Transactional(readOnly = true)`를 조회 메서드에 적용한다
- [ ] 단일 클래스 메서드 수 ≤ 10개
- [ ] `@RequiredArgsConstructor` + `final` 필드로 생성자 주입을 사용한다 (`@Autowired` 금지)

### 보안 코딩 (SC)

- [ ] MyBatis XML에서 `${}` 사용이 없다 (`#{}` 필수) — SC-001 SQL Injection
- [ ] ORDER BY 동적 정렬에 화이트리스트 검증을 적용한다 — SC-002
- [ ] 모든 공개 엔드포인트(로그인 제외)에 인증·인가가 적용되어 있다 — SC-003
- [ ] 비밀번호·토큰 등 민감 데이터가 로그·응답에 노출되지 않는다 — SC-004
- [ ] `@Valid` + Bean Validation 어노테이션으로 입력값을 검증한다 — SC-005

### 단위 테스트 커버리지 (CO)

- [ ] Service 레이어 테스트 커버리지 ≥ 80%
- [ ] Controller 레이어 테스트 커버리지 ≥ 70%
- [ ] 테스트 메서드에 `@DisplayName`으로 한국어 시나리오 설명이 있다
- [ ] given/when/then 또는 BDD 패턴이 일관되게 적용되어 있다
- [ ] 예외 케이스(NotFound, IllegalArgument 등)가 테스트에 포함되어 있다

### MyBatis/DBIO 규칙 (MB)

- [ ] Mapper XML `resultMap` 또는 `map-underscore-to-camel-case: true` 가 설정되어 있다
- [ ] N+1 문제가 발생하는 루프 내 Mapper 호출이 없다 (JOIN 또는 배치 조회 사용)
- [ ] 페이지 조회 쿼리에 `LIMIT/OFFSET` 또는 페이징 파라미터가 있다
- [ ] BXCM DBIO 방식 사용 시 인터페이스명이 `{domain}Dbio`다
- [ ] XML 위치 설정(`mapper-locations`)이 `application.yml`에 명시되어 있다

### 개발단계 검증 이슈 코드

| 코드 | 유형 | 심각도 |
|:---|:---|:---:|
| LC-001 | Controller → Mapper 직접 참조 | 높음 |
| LC-002 | Controller에 @Transactional 선언 | 높음 |
| LC-003 | @Autowired 필드 주입 사용 | 중간 |
| SC-001 | MyBatis ${} 사용 (SQL Injection) | 높음 |
| SC-002 | ORDER BY 동적 정렬 화이트리스트 미검증 | 높음 |
| SC-003 | 공개 엔드포인트 인증 누락 | 높음 |
| SC-004 | 민감 데이터 로그/응답 노출 | 높음 |
| SC-005 | 입력값 @Valid 검증 누락 | 중간 |
| CO-001 | Service 커버리지 < 80% | 중간 |
| CO-002 | 예외 케이스 테스트 누락 | 중간 |
| PF-001 | N+1 쿼리 발생 (루프 내 Mapper 호출) | 높음 |
| PF-002 | 페이지 조회 쿼리에 LIMIT 없음 | 중간 |

---

## 개발단계(프론트엔드-React) 품질 기준 (PLAN-006)

### TypeScript 타입 안전성 (TC)

- [ ] `any` 타입 사용 없음 (`: any`, `as any`, `<any>`) — TC-001
- [ ] 타입 단언(`as`) 남용 없음, Non-null assertion(`!`) 최소화 — TC-002
- [ ] null/undefined 처리에 optional chaining(`?.`) 사용 — TC-003
- [ ] 함수 반환 타입 명시 (추론 불가능한 경우) — TC-004
- [ ] Props 인터페이스 별도 정의 (인라인 타입 금지) — TC-005
- [ ] React 이벤트 핸들러 타입 명시 (`React.ChangeEvent<HTMLInputElement>` 등) — TC-007

### 레이어·컴포넌트 설계 (LC)

- [ ] 페이지 컴포넌트 150줄 이하 — LC-001
- [ ] API 호출이 Custom Hook으로 분리됨 (useEffect 직접 호출 금지) — LC-002
- [ ] Zustand store가 서버 상태(API 데이터) 관리하지 않음 — LC-003
- [ ] 인라인 스타일 미사용 (Tailwind 사용) — LC-004
- [ ] 컴포넌트 파일명 PascalCase — LC-005

### 성능 안티패턴 (PF)

- [ ] useEffect 의존성 배열에 누락 없음 (exhaustive-deps) — PF-001
- [ ] prop으로 전달하는 객체/배열/함수에 useMemo/useCallback 적용 — PF-002
- [ ] useMutation 후 invalidateQueries 호출로 캐시 갱신 — PF-004

### 보안 (SC)

- [ ] `dangerouslySetInnerHTML` 미사용 또는 sanitize 적용 — SC-001
- [ ] localStorage에 민감 데이터 평문 저장 없음 — SC-002
- [ ] 보호 라우트에 인증 체크 (`PrivateRoute` 또는 loader) 적용 — SC-003
- [ ] API URL 등 환경 변수가 소스코드에 하드코딩되지 않음 — SC-004

### 접근성 (A11Y)

- [ ] 이미지에 `alt` 속성 — A11Y-001
- [ ] 아이콘 전용 버튼에 `aria-label` — A11Y-002
- [ ] `<label>` htmlFor와 입력 필드 id 일치 — A11Y-003

### 개발단계(FE) 검증 이슈 코드

| 코드 | 유형 | 심각도 |
|:---|:---|:---:|
| TC-001 | any 타입 사용 | 높음 |
| TC-002 | 타입 단언 남용 | 중간 |
| TC-003 | null/undefined 처리 누락 | 중간 |
| LC-001 | 페이지 컴포넌트 150줄 초과 | 중간 |
| LC-002 | API 직접 호출 (Custom Hook 미분리) | 높음 |
| PF-001 | useEffect exhaustive-deps 위반 | 높음 |
| SC-001 | dangerouslySetInnerHTML 미검증 | 높음 |
| SC-002 | localStorage 민감 데이터 평문 저장 | 높음 |
| SC-003 | 보호 라우트 인증 누락 | 높음 |
| A11Y-002 | 아이콘 버튼 aria-label 누락 | 중간 |
| EV-001 | e2e: UC Happy Path 미존재 | 높음 |
| EV-003 | e2e: 하드코딩 선택자 사용 | 중간 |
| EV-006 | e2e: FORM 에러 시나리오 누락 | 중간 |

---

## 개발단계(프론트엔드-Next.js) 품질 기준 (PLAN-007)

Next.js App Router 고유 이슈 코드. fe-* 이슈 코드(TC/LC/PF/SC/A11Y)도 함께 적용한다.

### RSC/CC 경계 (NX)

- [ ] `'use client'`가 없는 파일은 Server Component임을 인지하고 설계 — NX-001
- [ ] CC에서 `prisma`, `fs` 등 서버 전용 모듈 직접 호출 없음 — NX-002
- [ ] `<img>` 태그 미사용, `next/image` 사용 — NX-003
- [ ] `<a href>` 태그 미사용, `next/link` 사용 — NX-004
- [ ] Route Handler에 `auth()` 인증 체크 포함 — NX-005
- [ ] Server Action에 Zod `safeParse` 유효성 검사 포함 — NX-006
- [ ] RSC `fetch()` 호출에 캐시 전략 명시 — NX-007
- [ ] `NEXT_PUBLIC_` 없는 env var를 CC에서 접근하지 않음 — NX-008
- [ ] Server Action 성공 후 `revalidatePath`/`revalidateTag` 호출 — NX-009
- [ ] 데이터 패칭 있는 Route Segment에 `error.tsx`·`loading.tsx` 존재 — NX-010

### 개발단계(Next.js) 검증 이슈 코드

| 코드 | 유형 | 심각도 |
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

---

## 개발단계(프론트엔드-Vue.js) 품질 기준 (PLAN-008)

Vue.js 3 + Vite 고유 이슈 코드. fe-* 이슈 코드(TC/PF/SC/A11Y)도 함께 적용한다.

### SFC 구조 (VV)

- [ ] `<script setup lang="ts">` 사용 — Options API 미사용 — VV-001/VV-002
- [ ] Pinia로 컴포넌트 간 상태 공유 (props drilling 2단계 초과 금지) — VV-003
- [ ] 복잡 로직은 Composable(`useXxx.ts`)로 분리 — VV-004
- [ ] `defineProps`/`defineEmits` 타입 제네릭 방식으로 명시 — VV-005
- [ ] `watchEffect` 대신 `watch` 사용 (의존성 명확화) — VV-006
- [ ] `$router`/`$route` 직접 접근 없음 — `useRouter()`/`useRoute()` 사용 — VV-007
- [ ] `v-for`에 `:key` 설정, index가 아닌 데이터 id 사용 — VV-008
- [ ] API 호출이 Composable + Vue Query로 분리 — VV-009
- [ ] 폼 submit에 VeeValidate + Zod 검증 적용 — VV-010

### 개발단계(Vue.js) 검증 이슈 코드

| 코드 | 유형 | 심각도 |
|:---|:---:|:---:|
| VV-001 | Options API 사용 — `<script setup>` 전환 권장 | 중간 |
| VV-002 | `<script setup>` 미사용 — `setup()` 반환 패턴 | 중간 |
| VV-003 | Pinia 없이 컴포넌트 간 상태 공유 (props drilling 2단계 초과) | 높음 |
| VV-004 | 컴포넌트 내 복잡 로직 — Composable 미추출 | 중간 |
| VV-005 | `defineProps`/`defineEmits` 타입 미정의 | 중간 |
| VV-006 | `watchEffect` 남용 — 의존성 불명확 | 낮음 |
| VV-007 | `$router`/`$route` 직접 접근 | 낮음 |
| VV-008 | `v-for` `:key` 미설정 또는 index 사용 | 높음 |
| VV-009 | 컴포넌트에서 API 직접 호출 — Vue Query 미사용 | 높음 |
| VV-010 | 폼 검증 없이 직접 submit — VeeValidate 미사용 | 높음 |
