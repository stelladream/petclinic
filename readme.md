# Spring PetClinic Sample Application

[![Java CI with Maven](https://github.com/spring-petclinic/spring-framework-petclinic/actions/workflows/maven-build.yml/badge.svg)](https://github.com/spring-petclinic/spring-framework-petclinic/actions/workflows/maven-build.yml)
[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=spring-petclinic_spring-framework-petclinic&metric=alert_status)](https://sonarcloud.io/dashboard?id=spring-petclinic_spring-framework-petclinic)
[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=spring-petclinic_spring-framework-petclinic&metric=coverage)](https://sonarcloud.io/dashboard?id=spring-petclinic_spring-framework-petclinic)

**This repo is a fork of the [spring-framework-petclinic](https://github.com/spring-petclinic/spring-framework-petclinic).
It allows the Spring commun**ity to maintain a Petclinic version with a plain old **Spring Framework configuration**
and with a **3-layer architecture** (i.e. presentation --> service --> repository).
The "canonical" implementation is now based on Spring Boot, Thymeleaf and [aggregate-oriented domain](https://github.com/spring-projects/spring-petclinic/pull/200).

## 기술 스택

| 항목 | 버전 |
|------|------|
| Spring Framework | 7.0.3 |
| Java | 21 |
| WAS | Tomcat 11 (Docker: `tomcat:11-jdk21`) |
| DB | MySQL 9 (Docker: `mysql:9.0`) |
| ORM | Hibernate 7 (JPA 3.2) |
| Hibernate Validator | 9.1.0.Final (Bean Validation 3.1) |
| Build | Maven |

## Understanding the Spring Petclinic application with a few diagrams

[See the presentation here](http://fr.slideshare.net/AntoineRey/spring-framework-petclinic-sample-application)

## 실행 방법

### 사전 요구사항

- Java 21
- Maven 3.9+
- Docker & Docker Compose

### 빌드 및 실행

```bash
# WAR 빌드
./mvnw clean package -P MySQL -DskipTests

# Docker Compose로 Tomcat 11 + MySQL 9 동시 기동
docker compose up -d

# 로그 확인
docker compose logs -f tomcat

# 종료 (볼륨 유지)
docker compose down

# 종료 + 볼륨 삭제 (DB 초기화)
docker compose down -v
```

접속: **http://localhost:8080**

<img width="1042" alt="petclinic-screenshot" src="https://cloud.githubusercontent.com/assets/838318/19727082/2aee6d6c-9b8e-11e6-81fe-e889a5ddfded.png">

> **포트 충돌 주의:** 로컬에 MySQL이 이미 3306 포트로 실행 중이면 `docker-compose.yml`의
> `"3306:3306"`을 `"3307:3306"`으로 변경 후 실행하세요.

## In case you find a bug/suggested improvement for Spring Petclinic

Our issue tracker is available here: https://github.com/spring-petclinic/spring-framework-petclinic/issues

## Database configuration

기본 설정은 H2 인메모리 DB를 사용합니다. **운영 권장 방식은 Docker Compose로 MySQL 9를 사용하는 것입니다.**

### Docker Compose (권장)

`docker-compose.yml`에 MySQL 9 + Tomcat 11이 구성되어 있습니다.

```bash
./mvnw clean package -P MySQL -DskipTests
docker compose up -d
```

MySQL 접속 정보 (컨테이너 내부):
```
host:     mysql (Docker 내부 네트워크)
port:     3306
database: petclinic
username: petclinic
password: petclinic
```

### 로컬 개발용 DB 프로파일

| 프로파일 | DB | 명령 |
|----------|----|------|
| H2 (기본) | 인메모리 | `./mvnw test` |
| MySQL | MySQL 9 | `-P MySQL` |
| PostgreSQL | PostgreSQL | `-P PostgreSQL` |

MySQL 프로파일 properties (`pom.xml`):

```xml
<properties>
    <jpa.database>MYSQL</jpa.database>
    <jdbc.driverClassName>com.mysql.cj.jdbc.Driver</jdbc.driverClassName>
    <jdbc.url>jdbc:mysql://localhost:3306/petclinic?serverTimezone=UTC</jdbc.url>
    <jdbc.username>petclinic</jdbc.username>
    <jdbc.password>petclinic</jdbc.password>
</properties>
```

PostgreSQL 프로파일 properties (`pom.xml`):

```xml
<properties>
    <jpa.database>POSTGRESQL</jpa.database>
    <jdbc.driverClassName>org.postgresql.Driver</jdbc.driverClassName>
    <jdbc.url>jdbc:postgresql://localhost:5432/petclinic</jdbc.url>
    <jdbc.username>postgres</jdbc.username>
    <jdbc.password>petclinic</jdbc.password>
</properties>
```

## Persistence layer choice

The persistence layer has 3 available implementations: **JPA** (default), **JDBC**, and **Spring Data JPA**.

Docker Compose 실행 시 `JAVA_TOOL_OPTIONS`의 `-Dspring.profiles.active=jpa`로 JPA가 기본 선택됩니다.
로컬 개발 시 `-Dspring.profiles.active` 옵션으로 변경할 수 있습니다:

| 프로파일 | 구현체 | 기술 |
|----------|--------|------|
| `jpa` (기본) | `repository/jpa/` | EntityManager + JPQL |
| `jdbc` | `repository/jdbc/` | NamedParameterJdbcTemplate, JdbcClient |
| `spring-data-jpa` | `repository/springdatajpa/` | Spring Data JPA |

## Compiling the CSS

There is a `petclinic.css` in `src/main/webapp/resources/css`.
It was generated from the `petclinic.scss` source, combined with the [Bootstrap](https://getbootstrap.com/) library.
If you make changes to the `scss`, or upgrade Bootstrap, you will need to re-compile the CSS resources
using the Maven profile "css":

```bash
./mvnw generate-resources -P css
```

## Working with Petclinic in your IDE

### Prerequisites

- Java 21 (full JDK)
- Maven 3.9+ (https://maven.apache.org/install.html)
- git (https://help.github.com/articles/set-up-git)
- Docker & Docker Compose
- IDE: IntelliJ IDEA / Eclipse (m2e) / Spring Tools Suite

### Steps

**1) 소스 클론**

```bash
git clone https://github.com/spring-petclinic/spring-framework-petclinic.git
cd spring-framework-petclinic
```

**2) Eclipse / Spring Tools Suite**

```
File -> Import -> Maven -> Existing Maven project
```

CSS 생성: `./mvnw generate-resources`
배포: `Run As -> Run on Server`로 Tomcat 11 서버에 배포

**3) IntelliJ IDEA**

`File > Open`에서 [pom.xml](pom.xml)을 선택해 프로젝트를 엽니다.

CSS 생성: `Maven -> Generates sources and Update Folders`

로컬 실행 옵션:
- `Run -> Edit Configuration`에서 Tomcat Server를 추가하고 `petclinic.war`를 배포
- 또는 Docker Compose 사용: `./mvnw clean package -P MySQL -DskipTests && docker compose up -d`

**4) 접속**

[http://localhost:8080](http://localhost:8080)

## Looking for something in particular?

| Java Config |   |
|-------------|---|
| Java config branch | Petclinic uses XML configuration by default. In case you'd like to use Java Config instead, there is a Java Config branch available [here](https://github.com/spring-petclinic/spring-framework-petclinic/tree/javaconfig) |

| Inside the 'Web' layer | Files |
|------------------------|-------|
| Spring MVC - XML integration | [mvc-view-config.xml](src/main/resources/spring/mvc-view-config.xml) |
| Spring MVC - ContentNegotiatingViewResolver | [mvc-view-config.xml](src/main/resources/spring/mvc-view-config.xml) |
| JSP custom tags | [WEB-INF/tags](src/main/webapp/WEB-INF/tags), [createOrUpdateOwnerForm.jsp](src/main/webapp/WEB-INF/jsp/owners/createOrUpdateOwnerForm.jsp) |
| JavaScript dependencies | [JavaScript libraries are declared as webjars in the pom.xml](pom.xml) |
| Static resources config | [Resource mapping in Spring configuration](src/main/resources/spring/mvc-core-config.xml) |
| Static resources usage | [htmlHeader.tag](src/main/webapp/WEB-INF/tags/htmlHeader.tag), [footer.tag](src/main/webapp/WEB-INF/tags/footer.tag) |

| 'Service' and 'Repository' layers | Files |
|-----------------------------------|-------|
| Transactions | [business-config.xml](src/main/resources/spring/business-config.xml), [ClinicServiceImpl.java](src/main/java/org/springframework/samples/petclinic/service/ClinicServiceImpl.java) |
| Cache | [tools-config.xml](src/main/resources/spring/tools-config.xml), [ClinicServiceImpl.java](src/main/java/org/springframework/samples/petclinic/service/ClinicServiceImpl.java) |
| Bean Profiles | [business-config.xml](src/main/resources/spring/business-config.xml), [ClinicServiceJdbcTests.java](src/test/java/org/springframework/samples/petclinic/service/ClinicServiceJdbcTests.java), [PetclinicInitializer.java](src/main/java/org/springframework/samples/petclinic/PetclinicInitializer.java) |
| JDBC | [business-config.xml](src/main/resources/spring/business-config.xml), [jdbc folder](src/main/java/org/springframework/samples/petclinic/repository/jdbc) |
| JPA | [business-config.xml](src/main/resources/spring/business-config.xml), [jpa folder](src/main/java/org/springframework/samples/petclinic/repository/jpa) |
| Spring Data JPA | [business-config.xml](src/main/resources/spring/business-config.xml), [springdatajpa folder](src/main/java/org/springframework/samples/petclinic/repository/springdatajpa) |

## Publishing a Docker image

This application uses [Google Jib](https://github.com/GoogleContainerTools/jib) to build an optimized Docker image
into the [Docker Hub](https://cloud.docker.com/u/springcommunity/repository/docker/springcommunity/spring-framework-petclinic/)
repository.

Jib containerizes this WAR project using `tomcat:11-jdk21` as the base image,
deploying the WAR to `/usr/local/tomcat/webapps/ROOT`.

```bash
./mvnw jib:build -P MySQL
```

## Interesting Spring Petclinic forks

The Spring Petclinic master branch in the main [spring-projects](https://github.com/spring-projects/spring-petclinic)
GitHub org is the "canonical" implementation, currently based on Spring Boot and Thymeleaf.

This [spring-framework-petclinic](https://github.com/spring-petclinic/spring-framework-petclinic) project is one of the [several forks](https://spring-petclinic.github.io/docs/forks.html)
hosted in a special GitHub org: [spring-petclinic](https://github.com/spring-petclinic).
If you have a special interest in a different technology stack
that could be used to implement the Pet Clinic then please join the community there.

## Interaction with other open source projects

| Name | Issue |
|------|-------|
| Spring JDBC: simplify usage of NamedParameterJdbcTemplate | [SPR-10256](https://github.com/spring-projects/spring-framework/issues/14889) and [SPR-10257](https://github.com/spring-projects/spring-framework/issues/14890) |
| Bean Validation / Hibernate Validator: simplify Maven dependencies and backward compatibility | [HV-790](https://hibernate.atlassian.net/browse/HV-790) and [HV-792](https://hibernate.atlassian.net/browse/HV-792) |
| Spring Data: provide more flexibility when working with JPQL queries | [DATAJPA-292](https://github.com/spring-projects/spring-data-jpa/issues/704) |

# Contributing

The [issue tracker](https://github.com/spring-petclinic/spring-framework-petclinic/issues) is the preferred channel for bug reports, features requests and submitting pull requests.

For pull requests, editor preferences are available in the [editor config](.editorconfig) for easy use in common text editors. Read more and download plugins at <http://editorconfig.org>.
