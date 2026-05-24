# Mapper 코드 템플릿 (MyBatis)

## Java Mapper 인터페이스

```java
package {{basePackage}}.mapper;

import {{basePackage}}.vo.{{도메인명}}VO;
import {{basePackage}}.vo.{{도메인명}}SearchVO;
import org.apache.ibatis.annotations.Mapper;
import java.util.List;

@Mapper
public interface {{도메인명}}Mapper {

    /** 목록 조회 */
    List<{{도메인명}}VO> selectList({{도메인명}}SearchVO searchVO);

    /** 전체 건수 */
    int count({{도메인명}}SearchVO searchVO);

    /** 단건 조회 */
    {{도메인명}}VO selectOne(Long {{pk필드명}});

    /** 등록 */
    int insert({{도메인명}}VO vo);

    /** 수정 */
    int update({{도메인명}}VO vo);

    /** 삭제 */
    int delete(Long {{pk필드명}});
}
```

---

## MyBatis XML 매핑 파일

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="{{basePackage}}.mapper.{{도메인명}}Mapper">

    <!-- ResultMap: 컬럼 → 필드 매핑 -->
    <resultMap id="{{도메인명}}ResultMap" type="{{basePackage}}.vo.{{도메인명}}VO">
        <id     column="{{pk컬럼명}}"   property="{{pk필드명}}"/>
        <result column="{{컬럼명1}}"    property="{{필드명1}}"/>
        <result column="{{컬럼명2}}"    property="{{필드명2}}"/>
        <result column="created_at"     property="createdAt"/>
        <result column="updated_at"     property="updatedAt"/>
        <result column="created_by"     property="createdBy"/>
    </resultMap>

    <!-- 공통 컬럼 SQL -->
    <sql id="columns">
        {{pk컬럼명}}, {{컬럼명1}}, {{컬럼명2}},
        created_at, updated_at, created_by
    </sql>

    <!-- 목록 조회 -->
    <select id="selectList" parameterType="{{basePackage}}.vo.{{도메인명}}SearchVO"
            resultMap="{{도메인명}}ResultMap">
        SELECT <include refid="columns"/>
        FROM   {{테이블명}}
        <where>
            <if test="keyword != null and keyword != ''">
                AND ({{컬럼명1}} LIKE CONCAT('%', #{keyword}, '%')
                  OR {{컬럼명2}} LIKE CONCAT('%', #{keyword}, '%'))
            </if>
            <if test="useYn != null and useYn != ''">
                AND use_yn = #{useYn}
            </if>
        </where>
        ORDER BY created_at DESC
        LIMIT #{size} OFFSET #{offset}
    </select>

    <!-- 전체 건수 -->
    <select id="count" parameterType="{{basePackage}}.vo.{{도메인명}}SearchVO"
            resultType="int">
        SELECT COUNT(*)
        FROM   {{테이블명}}
        <where>
            <if test="keyword != null and keyword != ''">
                AND ({{컬럼명1}} LIKE CONCAT('%', #{keyword}, '%')
                  OR {{컬럼명2}} LIKE CONCAT('%', #{keyword}, '%'))
            </if>
            <if test="useYn != null and useYn != ''">
                AND use_yn = #{useYn}
            </if>
        </where>
    </select>

    <!-- 단건 조회 -->
    <select id="selectOne" parameterType="long" resultMap="{{도메인명}}ResultMap">
        SELECT <include refid="columns"/>
        FROM   {{테이블명}}
        WHERE  {{pk컬럼명}} = #{{{pk필드명}}}
    </select>

    <!-- 등록 -->
    <insert id="insert" parameterType="{{basePackage}}.vo.{{도메인명}}VO"
            useGeneratedKeys="true" keyProperty="{{pk필드명}}">
        INSERT INTO {{테이블명}} (
            {{컬럼명1}},
            {{컬럼명2}},
            created_at,
            created_by
        ) VALUES (
            #{{{필드명1}}},
            #{{{필드명2}}},
            NOW(),
            #{createdBy}
        )
    </insert>

    <!-- 수정 -->
    <update id="update" parameterType="{{basePackage}}.vo.{{도메인명}}VO">
        UPDATE {{테이블명}}
        <set>
            <if test="{{필드명1}} != null">{{컬럼명1}} = #{{{필드명1}}},</if>
            <if test="{{필드명2}} != null">{{컬럼명2}} = #{{{필드명2}}},</if>
            updated_at = NOW()
        </set>
        WHERE {{pk컬럼명}} = #{{{pk필드명}}}
    </update>

    <!-- 삭제 -->
    <delete id="delete" parameterType="long">
        DELETE FROM {{테이블명}}
        WHERE {{pk컬럼명}} = #{{{pk필드명}}}
    </delete>

</mapper>
```

---

## foreach 동적 쿼리 예시 (IN 조건)

```xml
<select id="selectByIds" resultMap="{{도메인명}}ResultMap">
    SELECT <include refid="columns"/>
    FROM   {{테이블명}}
    WHERE  {{pk컬럼명}} IN
    <foreach collection="ids" item="id" open="(" separator="," close=")">
        #{id}
    </foreach>
</select>
```

---

## application.yml — MyBatis 설정

```yaml
mybatis:
  mapper-locations: classpath:mapper/**/*.xml
  type-aliases-package: {{basePackage}}.vo
  configuration:
    map-underscore-to-camel-case: true   # snake_case → camelCase 자동 변환
    default-fetch-size: 100
    default-statement-timeout: 30
    log-impl: org.apache.ibatis.logging.slf4j.Slf4jImpl
```
