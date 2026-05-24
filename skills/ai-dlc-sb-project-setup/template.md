# Spring Boot 프로젝트 설정 구조

## pom.xml 골격

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>{{spring_boot_version}}</version>
    </parent>
    <groupId>{{group_id}}</groupId>
    <artifactId>{{artifact_id}}</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>{{project_name}}</name>

    <properties>
        <java.version>{{java_version}}</java.version>
    </properties>

    <dependencies>
        <!-- Web -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <!-- MyBatis -->
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>3.0.3</version>
        </dependency>
        <!-- DB Driver: TODO: DB 종류에 맞게 교체 -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>
        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <!-- Validation -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        <!-- Test -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

---

## application.yml 골격

```yaml
# application.yml (공통 설정)
spring:
  profiles:
    active: dev
  application:
    name: {{project_name}}

server:
  port: 8080
  servlet:
    context-path: /

logging:
  level:
    root: INFO
    {{base_package}}: DEBUG
```

```yaml
# application-dev.yml (개발 환경)
spring:
  datasource:
    url: jdbc:mysql://localhost:3306/{{db_name}}?characterEncoding=UTF-8&serverTimezone=Asia/Seoul
    username: ${DB_USERNAME:devuser}
    password: ${DB_PASSWORD}        # TODO: 환경 변수로 관리
    driver-class-name: com.mysql.cj.jdbc.Driver
  mybatis:
    mapper-locations: classpath:mapper/**/*.xml
    configuration:
      map-underscore-to-camel-case: true
      log-impl: org.apache.ibatis.logging.stdout.StdOutImpl
```

```yaml
# application-prod.yml (운영 환경)
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
logging:
  level:
    root: WARN
    {{base_package}}: INFO
```

---

## 패키지 구조

```
src/
├── main/
│   ├── java/{{base_package}}/
│   │   ├── {{ProjectName}}Application.java
│   │   ├── config/           # Spring 설정 클래스
│   │   ├── controller/       # REST Controller
│   │   ├── service/          # Service 인터페이스 + Impl
│   │   ├── mapper/           # MyBatis Mapper 인터페이스
│   │   ├── vo/               # Value Object / DTO
│   │   └── exception/        # 예외 처리
│   └── resources/
│       ├── application.yml
│       ├── application-dev.yml
│       ├── application-stg.yml
│       ├── application-prod.yml
│       └── mapper/           # MyBatis XML 매핑 파일
└── test/
    └── java/{{base_package}}/
        ├── controller/
        └── service/
```
