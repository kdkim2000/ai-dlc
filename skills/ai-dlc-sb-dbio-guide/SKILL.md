---
name: ai-dlc-sb-dbio-guide
description: AI-DLC 개발단계(Spring Boot) 스킬. BXCM 기반 DBIO(Mapper) 작성 가이드라인을 제공한다. "DBIO 가이드", "BXCM Mapper 작성", "DBIO 사용법", "BXCM DBIO 규칙", "DBIO 어떻게 써", "BXCM 쿼리 작성", "DBIO 작성 방법" 같은 표현이 나오면 반드시 이 스킬을 사용하라.
allowed-tools: Read Grep Glob
---

# AI-DLC BXCM 기반 DBIO 작성 가이드라인

BXCM(Business eXecution Component Manager) 기반 DBIO(Database Input/Output) Mapper 작성 규칙·입출력 VO 매핑·쿼리 ID 채번 규칙을 대화창에 직접 출력한다. 파일을 저장하지 않는다.

## 트리거

- "DBIO 가이드", "BXCM Mapper 작성", "DBIO 사용법"
- "BXCM DBIO 규칙", "DBIO 어떻게 써", "BXCM 쿼리 작성"

---

## 가이드 내용 (인라인 내장)

### 1. DBIO 개요

DBIO는 BXCM 프레임워크에서 DB 접근을 처리하는 컴포넌트로, MyBatis 위에서 동작하는 래퍼 계층이다.

- **표준 인터페이스**: 입력 VO → DBIO → 출력 VO
- **쿼리 ID 기반 매핑**: XML에서 `queryId`로 SQL 구문 식별
- **단일 VO**: 입력과 출력을 하나의 VO로 통일하거나, ReqVO/ResVO로 분리

### 2. DBIO Mapper 인터페이스 패턴

```java
// BXCM DBIO 방식: interface에 @Mapper + 쿼리 ID 연계
@Mapper
public interface UserDbio {

    // 쿼리 ID: {namespace}.{동작}_{설명}
    List<UserVO> selectUserList(UserVO userVO);  // 목록 조회
    UserVO selectUserByPk(UserVO userVO);         // PK 단건 조회
    int insertUser(UserVO userVO);                // 등록
    int updateUser(UserVO userVO);                // 수정
    int deleteUser(UserVO userVO);                // 삭제 (물리)
    int updateUserUseYn(UserVO userVO);           // 논리 삭제
}
```

### 3. 쿼리 ID 채번 규칙

```
형식: {네임스페이스}.{동작}_{대상}_{조건}

예시:
  userDbio.selectUserList         — 사용자 목록 조회
  userDbio.selectUserByPk         — 사용자 PK 조회
  userDbio.selectUserByLoginId    — 로그인ID로 조회
  userDbio.insertUser             — 사용자 등록
  userDbio.updateUser             — 사용자 수정
  userDbio.deleteUser             — 사용자 삭제
  userDbio.updateUserUseYn        — 사용여부 변경
```

### 4. XML 매핑 파일 구조

```xml
<!-- src/main/resources/sql/UserDbio.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.example.mapper.UserDbio">

    <sql id="userColumns">
        user_id, user_nm, login_id, dept_cd,
        created_at, updated_at, created_by, use_yn
    </sql>

    <!-- 목록 조회 -->
    <select id="selectUserList" parameterType="UserVO" resultType="UserVO">
        SELECT <include refid="userColumns"/>
        FROM   TB_USER
        <where>
            AND use_yn = 'Y'
            <if test="userNm != null and userNm != ''">
                AND user_nm LIKE CONCAT('%', #{userNm}, '%')
            </if>
            <if test="deptCd != null and deptCd != ''">
                AND dept_cd = #{deptCd}
            </if>
        </where>
        ORDER BY created_at DESC
        LIMIT #{pageSize} OFFSET #{offset}
    </select>

    <!-- 등록 -->
    <insert id="insertUser" parameterType="UserVO"
            useGeneratedKeys="true" keyProperty="userId">
        INSERT INTO TB_USER (
            user_nm, login_id, dept_cd, created_at, created_by, use_yn
        ) VALUES (
            #{userNm}, #{loginId}, #{deptCd}, NOW(), #{createdBy}, 'Y'
        )
    </insert>

</mapper>
```

### 5. 입출력 VO 매핑 원칙

```java
// BXCM DBIO 방식: 단일 VO로 입출력 통일
public class UserVO {
    // 입력 필드 (조회 조건)
    private String userNm;
    private String deptCd;
    private int pageSize;
    private int offset;

    // 출력 필드 (조회 결과)
    private Long userId;
    private String loginId;
    private LocalDateTime createdAt;
    private String useYn;
}
```

### 6. DBIO XML 파일 위치 설정

```yaml
# application.yml
mybatis:
  mapper-locations:
    - classpath:mapper/**/*.xml
    - classpath:sql/**/*.xml    # DBIO XML 위치 추가
```

### 7. 일반 MyBatis vs BXCM DBIO 비교

| 항목 | 일반 MyBatis | BXCM DBIO |
|:---|:---|:---|
| 인터페이스 명 | `{도메인명}Mapper` | `{도메인명}Dbio` |
| XML 위치 | `resources/mapper/` | `resources/sql/` |
| 파라미터 | 전용 SearchVO | 통합 VO 1개 |
| 쿼리 ID | 메서드명과 동일 | `namespace.동작_설명` |
| 페이징 | SearchVO에 page/size | VO에 pageSize/offset |
