# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

```bash
# Run application (default: H2 in-memory DB with JPA)
./mvnw jetty:run-war

# Run with a specific database
./mvnw jetty:run-war -P MySQL
./mvnw jetty:run-war -P PostgreSQL

# Run with a specific persistence layer
./mvnw jetty:run-war -Dspring.profiles.active=jdbc
./mvnw jetty:run-war -Dspring.profiles.active=spring-data-jpa

# Build
./mvnw clean install

# Run all tests
./mvnw test

# Run a single test class
./mvnw test -Dtest=ClinicServiceJpaTests

# Run a single test method
./mvnw test -Dtest=ClinicServiceJpaTests#shouldFindOwnersByLastName

# Compile CSS from SCSS (only needed if modifying styles)
./mvnw generate-resources -P css
```

Test files must match the pattern `**/*Tests.java` (Surefire convention).

## Architecture

This is a **Spring Framework 7** web application (WAR) using **XML-based Spring configuration** — not Spring Boot, not annotation-based Java config. All bean wiring is in `src/main/resources/spring/*.xml`.

### Three-Layer Structure

```
web/        → @Controller classes (Spring MVC, JSP views)
service/    → ClinicService facade (@Transactional, @Cacheable)
repository/ → Data access interfaces with pluggable implementations
model/      → JPA entities with Jakarta Validation
```

All controllers depend on a single `ClinicService` interface, which delegates to four repository interfaces (`OwnerRepository`, `PetRepository`, `VetRepository`, `VisitRepository`).

### Pluggable Persistence (Spring Profiles)

Three repository implementations are selected at runtime via Spring profiles:

| Profile | Implementation | Technology |
|---|---|---|
| `jpa` (default) | `repository/jpa/` | EntityManager + JPQL |
| `jdbc` | `repository/jdbc/` | NamedParameterJdbcTemplate, JdbcClient |
| `spring-data-jpa` | `repository/springdatajpa/` | Spring Data JPA interfaces |

Database profiles (`H2` default, `HSQLDB`, `MySQL`, `PostgreSQL`) are independent — combine them: `-P MySQL -Dspring.profiles.active=jdbc`.

### Spring Configuration Files (`src/main/resources/spring/`)

- **business-config.xml** — Component scanning for service/repository, profile-specific bean definitions, transaction management
- **datasource-config.xml** — DataSource, connection pool, DB init scripts
- **mvc-core-config.xml** — Controller scanning, static resources, exception handling
- **mvc-view-config.xml** — Content negotiation (JSON/XML/HTML), JSP view resolver
- **tools-config.xml** — AOP call monitoring, Caffeine cache manager

### Entry Point

`PetclinicInitializer` extends `AbstractDispatcherServletInitializer` (Servlet 3.0+ programmatic config, no web.xml). It loads the XML configs and sets the default profile to `jpa`.

### Key Patterns

- **Constructor injection** throughout (no field injection)
- **Entity hierarchy**: `BaseEntity` → `NamedEntity` / `Person` → domain classes
- **Caching**: Caffeine cache on `ClinicService.findVets()` via `@Cacheable("vets")`
- **Validation**: Jakarta Bean Validation annotations on entities + custom `PetValidator`
- **Formatting**: `PetTypeFormatter` for Spring data binding of `PetType`

## Code Style

- Java 17, UTF-8, LF line endings, 4-space indentation (see `.editorconfig`)
