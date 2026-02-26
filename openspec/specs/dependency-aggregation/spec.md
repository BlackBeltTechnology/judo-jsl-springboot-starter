# dependency-aggregation Specification

## Purpose

This project serves as a Spring Boot starter that aggregates all JUDO runtime, DAO, code generation, and supporting dependencies into a single Maven artifact (`judo-spring-boot-starter`), so that generated JSL-based Spring Boot applications can declare one dependency instead of managing dozens individually.

## Architecture

The project is a single `pom.xml` with `jar` packaging that declares direct dependencies on three major groups:
- **Spring Boot starters** (`spring-boot-starter`, `spring-boot-starter-data-jdbc`) managed by Spring Boot BOM 3.5.0
- **JUDO Runtime Core** (runtime, spring integration, dispatcher, access manager, DAO layers for HSQLDB and PostgreSQL) managed by `judo-runtime-core-dependencies` BOM
- **JUDO PSM Generator SDK** (`sdk-core-common`, `-api`, `-impl`, `-spring`) with explicit version properties

Version management uses `${revision}` property (currently `1.0.4-SNAPSHOT`) flattened by `flatten-maven-plugin` at build time. Dependency versions for JUDO components are controlled via `judo-runtime-core-version` and `judo-psm-generator-sdk-core-version` properties in the POM.

## Requirements

### Requirement: Starter SHALL provide all JUDO runtime dependencies
The starter artifact SHALL transitively provide the complete JUDO runtime stack so that generated projects need only declare a single dependency.

#### Scenario: Generated project depends on starter
- **GIVEN** a Maven project with `judo-spring-boot-starter` as a dependency
- **WHEN** the project resolves its dependency tree
- **THEN** all JUDO runtime core, DAO, dispatcher, access manager, and Spring integration artifacts are available on the classpath

### Requirement: Starter SHALL provide PSM Generator SDK
The starter SHALL include the JUDO PSM Generator SDK artifacts needed for code generation in consuming projects.

#### Scenario: PSM generator classes available
- **GIVEN** a Maven project with `judo-spring-boot-starter` as a dependency
- **WHEN** the project attempts to use `judo-psm-generator-sdk-core-api` classes
- **THEN** the classes are available without declaring an additional dependency

### Requirement: Starter SHALL manage Spring Boot version
The starter SHALL import the Spring Boot BOM to ensure consistent Spring dependency versions across all consuming projects.

#### Scenario: Spring Boot version consistency
- **GIVEN** a Maven project with `judo-spring-boot-starter` as a dependency
- **WHEN** the project resolves Spring-related transitive dependencies
- **THEN** all Spring dependencies resolve to versions consistent with Spring Boot 3.5.0

### Requirement: Starter SHALL support both HSQLDB and PostgreSQL
The starter SHALL include DAO RDBMS implementations and Spring configuration support for both HSQLDB (development/testing) and PostgreSQL (production).

#### Scenario: HSQLDB available for testing
- **GIVEN** a Maven project with `judo-spring-boot-starter` as a dependency
- **WHEN** the project configures HSQLDB as its database
- **THEN** `judo-runtime-core-dao-rdbms-hsqldb` and `judo-runtime-core-spring-hsqldb` are available

#### Scenario: PostgreSQL available for production
- **GIVEN** a Maven project with `judo-spring-boot-starter` as a dependency
- **WHEN** the project configures PostgreSQL as its database
- **THEN** `judo-runtime-core-dao-rdbms-postgresql` and `judo-runtime-core-spring-postgresql` are available

### Requirement: Published POM SHALL have resolved version
The `flatten-maven-plugin` SHALL produce a flattened POM where `${revision}` is replaced with the actual version string before publishing.

#### Scenario: Flattened POM in repository
- **GIVEN** the project is built with `mvn install`
- **WHEN** the artifact is installed to the local repository
- **THEN** the published POM contains a literal version string (e.g., `1.0.4-SNAPSHOT`), not `${revision}`

### Requirement: ANTLR runtime SHALL be pinned to 3.2
The starter SHALL explicitly declare `antlr-runtime` 3.2 to prevent transitive dependency resolution from upgrading to an incompatible version (Eclipse Xtext 2.27.0 requires < 3.5).

#### Scenario: ANTLR version override
- **GIVEN** a Maven project with `judo-spring-boot-starter` as a dependency
- **WHEN** the dependency tree resolves `org.antlr:antlr-runtime`
- **THEN** the resolved version is 3.2, regardless of what transitive dependencies request
