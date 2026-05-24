---
name: ai-dlc-sb-db-guide
description: AI-DLC 개발단계(Spring Boot) 스킬. 데이터베이스 개발 가이드라인을 제공한다. "DB 개발 가이드", "데이터베이스 개발 규칙", "쿼리 작성 규칙", "DB 코딩 표준", "DB 성능 가이드", "인덱스 설계", "트랜잭션 관리" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC 데이터베이스 개발 가이드라인

인덱스 사용 규칙·쿼리 최적화·트랜잭션 범위·커넥션 풀·배치 처리 기준을 대화창에 직접 출력한다. 파일을 저장하지 않는다.

## 트리거

- "DB 개발 가이드", "데이터베이스 개발 규칙", "쿼리 작성 규칙"
- "DB 코딩 표준", "DB 성능 가이드", "인덱스 설계"

---

## 가이드 내용 (인라인 내장)

### 1. 인덱스 설계 원칙

```sql
-- FK 컬럼에 인덱스 필수
CREATE INDEX IX_TB_ORDER_USER_ID ON TB_ORDER (user_id);

-- 자주 조회되는 조건 컬럼
CREATE INDEX IX_TB_USER_DEPT_CD ON TB_USER (dept_cd, use_yn);

-- 복합 인덱스 순서: 카디널리티 높은 컬럼 → 낮은 컬럼
CREATE INDEX IX_TB_ORDER_STATUS_DATE ON TB_ORDER (order_status, order_date);

-- 유니크 제약 (이메일, 로그인 ID)
CREATE UNIQUE INDEX UIX_TB_USER_LOGIN_ID ON TB_USER (login_id);
```

**규칙**:
- WHERE 절에 자주 사용되는 컬럼에 인덱스 생성
- `LIKE '%키워드%'` 앞에 `%`가 있으면 인덱스 효과 없음 → Full-Text Index 고려
- 인덱스가 많을수록 INSERT/UPDATE 성능 저하 → 필요한 것만

### 2. 쿼리 최적화

```sql
-- 나쁜 예: 인덱스 무력화
WHERE DATE_FORMAT(created_at, '%Y-%m') = '2024-01'  -- 함수 적용 → 인덱스 무효

-- 좋은 예: 범위 조건으로 변경
WHERE created_at >= '2024-01-01' AND created_at < '2024-02-01'

-- 나쁜 예: SELECT *
SELECT * FROM TB_USER WHERE dept_cd = 'D001'

-- 좋은 예: 필요 컬럼만
SELECT user_id, user_nm, email FROM TB_USER WHERE dept_cd = 'D001'

-- 나쁜 예: 대용량 IN
WHERE user_id IN (SELECT user_id FROM TB_TEMP_TABLE)  -- 서브쿼리

-- 좋은 예: JOIN으로 변경
FROM TB_USER u INNER JOIN TB_TEMP_TABLE t ON u.user_id = t.user_id
```

### 3. 트랜잭션 범위

```java
// 올바른 트랜잭션 범위: Service 레이어
@Service
public class OrderServiceImpl implements OrderService {

    @Transactional  // 등록/수정/삭제
    public void createOrder(OrderVO vo) {
        orderMapper.insert(vo);
        inventoryMapper.decreaseStock(vo.getProductId(), vo.getQuantity());
    }

    @Transactional(readOnly = true)  // 조회 전용 (Lock 미획득)
    public OrderVO getOrder(Long orderId) {
        return orderMapper.selectOne(orderId);
    }
}

// 금지: Controller에 @Transactional
@Transactional  // Controller에 선언 금지
@PostMapping
public ResponseEntity<?> create(...) { ... }
```

**트랜잭션 격리 수준** (MySQL InnoDB 기본: REPEATABLE_READ):
- 일반 조회: `readOnly = true` 사용으로 성능 최적화
- 재고·잔액 처리: `@Transactional(isolation = Isolation.READ_COMMITTED)` 고려

### 4. 커넥션 풀 설정 (HikariCP)

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 10       # CPU 코어 수 × 2 + 디스크 수
      minimum-idle: 5
      connection-timeout: 30000   # 30초 (연결 대기 시간)
      idle-timeout: 600000        # 10분 (유휴 커넥션 반환)
      max-lifetime: 1800000       # 30분 (커넥션 최대 수명)
      connection-test-query: SELECT 1
```

### 5. 배치 처리 기준

```java
// 1000건 이상: 배치 처리
@Transactional
public void batchInsert(List<UserVO> users) {
    int batchSize = 500;
    for (int i = 0; i < users.size(); i += batchSize) {
        List<UserVO> batch = users.subList(i, Math.min(i + batchSize, users.size()));
        userMapper.batchInsert(batch);
    }
}
```

```xml
<!-- MyBatis 배치 INSERT -->
<insert id="batchInsert" parameterType="list">
    INSERT INTO TB_USER (user_nm, email, created_at) VALUES
    <foreach collection="list" item="vo" separator=",">
        (#{vo.userNm}, #{vo.email}, NOW())
    </foreach>
</insert>
```

**기준**:
- 건수 < 100: 단건 반복
- 건수 100~10000: 배치 INSERT (foreach)
- 건수 > 10000: Spring Batch 또는 LOAD DATA 고려

### 6. 명명 규칙

| 대상 | 규칙 | 예시 |
|:---|:---|:---|
| 테이블 | `TB_{업무}_{대상}` (대문자) | `TB_USER`, `TB_ORDER_DETAIL` |
| 컬럼 | snake_case, 약어 최소화 | `user_nm`, `created_at` |
| PK | `{테이블명_단수}_id` | `user_id`, `order_id` |
| FK | 참조 테이블 PK와 동일 | `user_id` (TB_ORDER.user_id) |
| 인덱스 | `IX_{테이블명}_{컬럼}` | `IX_TB_ORDER_USER_ID` |
