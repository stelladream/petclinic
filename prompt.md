# 작업 목표
https://github.com/spring-petclinic/spring-framework-petclinic 프로젝트를 클론한 상태에서,
아래 스펙으로 전환하는 모든 수정 작업을 수행해줘.

## 목표 스펙
- Spring Framework: 6.2.x → 7.0.2
- Java: 기존 → 21 (tomcat:11-jdk21 런타임과 일치시켜야 함)
- WAS: Jetty → Docker Tomcat 11 (tomcat:11-jdk21 이미지)
- DB: H2(기본) → Docker MySQL 9 (mysql:9.0 이미지)
- 빌드: Maven (기존 유지)
- 실행 방식: docker compose up 으로 Tomcat + MySQL 동시 기동

---

## 수행할 작업 목록

### 1. pom.xml 수정

다음 버전 프로퍼티를 변경해줘:
- spring-framework.version → 7.0.2
- java.version → 21 (tomcat:11-jdk21 런타임 기준. 25로 설정하면 컴파일된 바이트코드를 Java 21 JVM이 로드 불가)
- hibernate.version → 7.0.1.Final (Hibernate ORM 7, JPA 3.2 대응)
- mysql-connector 버전 → com.mysql:mysql-connector-j:9.1.0
- jakarta.servlet-api 버전 → 6.1.0
- jakarta.servlet.jsp-api 버전 → 4.0.0
- hibernate-validator 버전 → 9.0.0.Final (Bean Validation 3.1 대응)
- JSTL → org.glassfish.web:jakarta.servlet.jsp.jstl:3.0.1

Maven Compiler Plugin의 `<release>`를 `${java.version}`으로 설정해줘 (하드코딩 대신 프로퍼티 참조).

Jetty Maven Plugin은 삭제해줘. Tomcat Maven Plugin은 Maven Central에 Tomcat 10+ 지원 버전이 없으므로
추가하지 말고, 로컬 실행은 IDE(IntelliJ/Eclipse)의 Tomcat 서버 통합 기능을 사용해줘.

Maven 리소스 필터링 설정을 아래와 같이 변경해줘 (Spring XML 파일은 필터링에서 제외):
```xml
<resources>
    <resource>
        <directory>${basedir}/src/main/resources</directory>
        <filtering>true</filtering>
        <excludes>
            <exclude>spring/*.xml</exclude>
        </excludes>
    </resource>
    <resource>
        <directory>${basedir}/src/main/resources</directory>
        <filtering>false</filtering>
        <includes>
            <include>spring/*.xml</include>
        </includes>
    </resource>
</resources>
```
> 이유: Maven 필터링이 Spring XML의 `${jdbc.url}` 플레이스홀더를 빌드 시점에 치환하면,
> URL에 포함된 `&`가 XML 엔티티로 파싱되어 `SAXParseException`이 발생하고,
> 런타임 시스템 프로퍼티 오버라이드도 불가능해짐.

MySQL 프로파일(id: MySQL)의 properties를 아래와 같이 수정해줘:
- jpa.database: MYSQL
- jdbc.driverClassName: com.mysql.cj.jdbc.Driver
- jdbc.url: jdbc:mysql://localhost:3306/petclinic?serverTimezone=UTC
- jdbc.username: petclinic
- jdbc.password: petclinic

> 주의: URL에 `useUnicode=true&serverTimezone=UTC`처럼 `&`를 사용하면 Maven XML 파싱 오류 발생.
> `useUnicode=true`는 MySQL Connector/J 9.x 기본값이므로 생략 가능.
> 파라미터가 여러 개 필요한 경우 pom.xml에서는 `&amp;amp;`로 이중 이스케이프해야 함.

javax.* 관련 의존성(javax.servlet:servlet-api, javax.annotation:*, javax.inject:*)이
있다면 전부 삭제해줘.

### 2. web.xml 수정

src/main/webapp/WEB-INF/web.xml의 Servlet 버전 선언을 6.1로 올려줘:
- namespace: https://jakarta.ee/xml/ns/jakartaee
- schemaLocation: web-app_6_1.xsd
- version="6.1"

### 3. Java 소스 코드 확인 및 수정

src/main/java 전체를 grep해서 아래 패키지가 남아있으면 jakarta로 교체해줘:
- javax.annotation.* → jakarta.annotation.*
- javax.inject.* → jakarta.inject.*
- javax.persistence.* → jakarta.persistence.*
- javax.validation.* → jakarta.validation.*
- javax.servlet.* → jakarta.servlet.*

> javax.sql.DataSource는 Java SE 표준이므로 교체 대상이 아님.

### 4. Spring XML 설정 파일 수정

src/main/resources/spring/business-config.xml에서
Hibernate dialect를 아래와 같이 수정해줘:
- 기존: org.hibernate.dialect.MySQL8Dialect (또는 유사 이름)
- 변경: org.hibernate.dialect.MySQLDialect

JPA properties에 아래 항목도 확인하고 추가해줘:
- hibernate.dialect: org.hibernate.dialect.MySQLDialect
- hibernate.hbm2ddl.auto: update (개발 편의)

> 주의: hibernate.dialect를 MySQLDialect로 하드코딩하면 H2/HSQLDB 프로파일 테스트가 깨짐.
> HibernateJpaVendorAdapter의 p:database="${jpa.database}" 설정이 dialect를 자동 선택하므로,
> 별도 dialect 프로퍼티 없이도 동작함. jpaProperties 추가는 선택사항으로 처리해줘.

### 5. Jib Plugin 수정 (Docker 이미지 빌드용)

pom.xml의 jib-maven-plugin에서:
- `<from><image>`를 tomcat:11-jdk21 로 변경
- `<container><appRoot>`를 /usr/local/tomcat/webapps/ROOT 로 변경
- Jetty용 `<entrypoint>` 설정은 삭제

### 6. docker-compose.yml 신규 생성 (프로젝트 루트에)

아래 내용으로 docker-compose.yml을 생성해줘:
```yaml
services:
  mysql:
    image: mysql:9.0
    container_name: petclinic-mysql
    environment:
      MYSQL_ROOT_PASSWORD: root
      MYSQL_DATABASE: petclinic
      MYSQL_USER: petclinic
      MYSQL_PASSWORD: petclinic
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost", "-upetclinic", "-ppetclinic"]
      interval: 10s
      timeout: 5s
      retries: 10
      start_period: 30s

  tomcat:
    image: tomcat:11-jdk21
    container_name: petclinic-tomcat
    ports:
      - "8080:8080"
    volumes:
      - ./target/petclinic.war:/usr/local/tomcat/webapps/ROOT.war
    depends_on:
      mysql:
        condition: service_healthy
    environment:
      JAVA_TOOL_OPTIONS: >-
        -Dspring.profiles.active=jpa
        -Djdbc.url=jdbc:mysql://mysql:3306/petclinic?serverTimezone=UTC
        -Djdbc.username=petclinic
        -Djdbc.password=petclinic
        -Djpa.database=MYSQL
        -Djdbc.driverClassName=com.mysql.cj.jdbc.Driver

volumes:
  mysql_data:
```

> 주의1: `version: '3.9'`는 최신 Docker Compose에서 deprecated이므로 생략.
> 주의2: `JAVA_OPTS` 대신 `JAVA_TOOL_OPTIONS`를 사용해야 함.
>   Tomcat의 catalina.sh는 JAVA_OPTS를 `eval`로 실행하므로 값에 포함된 `&`가
>   쉘 백그라운드 연산자로 해석되어 `-Djdbc.username=petclinic: command not found` 오류 발생.
>   `JAVA_TOOL_OPTIONS`는 JVM이 쉘을 거치지 않고 직접 읽으므로 `&`가 안전하게 처리됨.
> 주의3: 로컬에 MySQL이 3306으로 이미 실행 중이면 포트 충돌 발생. 필요시 "3307:3306"으로 변경.

### 7. README.md 수정

기존 Jetty 실행 명령 섹션을 아래 내용으로 교체해줘:
```markdown
## 실행 방법

### 사전 요구사항
- Java 21
- Maven 3.9+
- Docker & Docker Compose

### 빌드 및 실행

# WAR 빌드
./mvnw clean package -P MySQL -DskipTests

# Docker Compose로 Tomcat 11 + MySQL 9 동시 기동
docker compose up -d

# 로그 확인
docker compose logs -f

# 종료
docker compose down
```

접속: http://localhost:8080

---

## 완료 후 검증 요청

모든 수정이 끝나면 아래 항목을 직접 확인하고 결과를 알려줘:

1. `./mvnw dependency:resolve -P MySQL` 실행해서 의존성 오류가 없는지 확인
2. `grep -r "javax\." src/main/java/` 실행해서 javax 잔재가 없는지 확인 (javax.sql 제외)
3. `./mvnw clean package -P MySQL -DskipTests` 빌드가 성공하는지 확인
4. 빌드 성공 시 target/petclinic.war 파일이 생성됐는지 확인
5. `docker compose up -d` 실행 후 `docker compose logs tomcat`으로 정상 기동 확인
6. `curl http://localhost:8080/vets` 로 데이터 정상 조회 확인
7. 변경된 파일 목록을 최종 정리해서 보고해줘

## 주의사항
- pom.xml에서 Spring Framework BOM을 사용한다면 버전 override 시 충돌이 없는지 확인
- Tomcat 11은 Jakarta EE 11 (Servlet 6.1) 기반이므로 Servlet 6.0 API와 혼용 금지
- mysql-connector-j 9.x는 com.mysql groupId를 사용 (mysql groupId 아님)
- Hibernate 7은 javax.persistence 패키지를 완전히 제거했으므로 jakarta.persistence만 사용
- java.version은 반드시 Docker Tomcat 이미지의 JDK 버전과 일치시킬 것
  (tomcat:11-jdk21 → java.version=21, tomcat:11-jdk25 → java.version=25)
- Spring XML 파일(src/main/resources/spring/*.xml)은 Maven 리소스 필터링에서 반드시 제외할 것
  (미제외 시 ${jdbc.url} 등 Spring 플레이스홀더가 빌드 시점에 치환되어 XML 파싱 오류 및 런타임 오버라이드 불가)
- JAVA_TOOL_OPTIONS 사용 (JAVA_OPTS X): catalina.sh의 eval이 & 문자를 쉘 연산자로 해석함
- tomcat10-maven-plugin은 Maven Central에 배포되지 않아 사용 불가
