---
name: ai-dlc-sb-mybatis-guide
description: AI-DLC 개발단계(Spring Boot) 스킬. MyBatis 쿼리 작성 가이드라인을 제공한다. "MyBatis 가이드", "MyBatis 쿼리 작성법", "매퍼 XML 작성", "MyBatis 동적 쿼리", "MyBatis 어떻게 써", "매퍼 작성 규칙", "MyBatis 설정 방법" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC MyBatis 쿼리 작성 가이드라인

MyBatis 3.x 기반 Mapper 인터페이스·XML 동적 쿼리·ResultMap·N+1 방지 패턴을 대화창에 직접 출력한다. 파일을 저장하지 않는다.

## 트리거

- "MyBatis 가이드", "MyBatis 쿼리 작성법", "매퍼 XML 작성"
- "MyBatis 동적 쿼리", "MyBatis 어떻게 써", "매퍼 작성 규칙"

---

## 가이드 내용 (인라인 내장)

### 1. Mapper 인터페이스 패턴

```java
@Mapper
public interface UserMapper {
    List<UserVO> selectList(UserSearchVO searchVO);
    UserVO selectOne(Long userId);
    int insert(UserVO vo);
    int update(UserVO vo);
    int delete(Long userId);
}
```

**규칙**:
- `@Mapper` 어노테이션 필수 (또는 `@MapperScan` 설정)
- 메서드명과 XML id 반드시 일치
- 파라미터가 2개 이상이면 `@Param` 사용 또는 VO로 묶기

### 2. 파라미터 바인딩 규칙

```xml
<!-- 올바름: #{} → PreparedStatement, SQL Injection 방지 -->
WHERE user_nm LIKE CONCAT('%', #{keyword}, '%')

<!-- 금지: ${} → Statement, SQL Injection 위험 -->
WHERE user_nm LIKE '%${keyword}%'  <!-- 절대 사용 금지 -->

<!-- 정렬 컬럼: 화이트리스트 검증 후 ${}만 허용 -->
ORDER BY ${sortBy} ${sortDir}
<!-- 반드시 서비스 레이어에서 허용 컬럼 목록 검증 필요 -->
```

### 3. 동적 쿼리 패턴

```xml
<!-- where 태그: 첫 번째 AND/OR 자동 제거 -->
<where>
    <if test="keyword != null and keyword != ''">
        AND user_nm LIKE CONCAT('%', #{keyword}, '%')
    </if>
    <if test="useYn != null">
        AND use_yn = #{useYn}
    </if>
</where>

<!-- set 태그: 마지막 쉼표 자동 제거 -->
<set>
    <if test="userNm != null">user_nm = #{userNm},</if>
    <if test="email != null">email = #{email},</if>
    updated_at = NOW()
</set>

<!-- foreach: IN 조건 -->
WHERE user_id IN
<foreach collection="ids" item="id" open="(" separator="," close=")">
    #{id}
</foreach>

<!-- choose/when/otherwise: switch-case -->
<choose>
    <when test="type == 'A'">AND type_cd = 'A'</when>
    <when test="type == 'B'">AND type_cd = 'B'</when>
    <otherwise>AND type_cd IS NOT NULL</otherwise>
</choose>
```

### 4. ResultMap — 컬럼-필드 매핑

```xml
<resultMap id="UserResultMap" type="com.example.vo.UserVO">
    <id     column="user_id"   property="userId"/>
    <result column="user_nm"   property="userNm"/>
    <result column="created_at" property="createdAt"/>
    <!-- 1:N 관계 -->
    <collection property="roles" ofType="RoleVO"
                select="selectRolesByUserId" column="user_id"/>
</resultMap>
```

**설정 단축**: `map-underscore-to-camel-case: true` 설정 시 ResultMap 없이 snake_case → camelCase 자동 변환

### 5. N+1 쿼리 방지

```xml
<!-- N+1 발생 패턴: 목록 조회 후 각 건마다 별도 쿼리 -->
<!-- 개선: JOIN으로 한 번에 조회 -->
<select id="selectListWithDetail" resultMap="UserWithDetailMap">
    SELECT u.*, d.dept_nm
    FROM   TB_USER u
    LEFT JOIN TB_DEPT d ON u.dept_cd = d.dept_cd
    <where>
        <if test="keyword != null">AND u.user_nm LIKE CONCAT('%', #{keyword}, '%')</if>
    </where>
</select>
```

### 6. application.yml MyBatis 설정

```yaml
mybatis:
  mapper-locations: classpath:mapper/**/*.xml
  type-aliases-package: com.example.vo
  configuration:
    map-underscore-to-camel-case: true
    default-fetch-size: 100
    default-statement-timeout: 30
    cache-enabled: false  # 2차 캐시 비활성화 (멀티 인스턴스 환경)
```

### 7. 공통 실수 방지

| 실수 | 원인 | 해결 |
|:---|:---|:---|
| `${}` 사용 | SQL Injection 위험 | `#{}` 로 전환 |
| resultMap 미사용 | 타입 불일치 오류 | resultMap 명시 |
| collection lazy 미설정 | N+1 발생 | JOIN 쿼리로 변경 |
| 파라미터 타입 불일치 | `Long` vs `long` | VO 필드 타입 확인 |
