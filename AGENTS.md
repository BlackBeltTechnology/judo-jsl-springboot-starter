# JUDO JSL Spring Boot Starter - Project Documentation

## Project Overview


**Repository:** BlackBeltTechnology/judo-jsl-springboot-starter
**License:** Eclipse Public License 2.0 (EPL-2.0)
**Java Version:** 21 (Zulu distribution in CI)
**Build System:** Maven 3.8.6 with Maven Wrapper (`./mvnw`)

1. This is a **Spring Boot starter/dependency aggregator** — it bundles the entire JUDO runtime stack into a single Maven artifact for generated JSL-based applications
2. The project contains **no application source code** — only a `pom.xml` defining dependencies, build plugins, and Maven profiles
3. It provides Spring Boot 3.5.0 integration, JUDO runtime core (dispatcher, DAO, access management), PSM generator SDK, and database support for HSQLDB and PostgreSQL
4. Generated JUDO projects declare this starter as a dependency to inherit all required runtime and code-generation libraries
5. Versioning uses `${revision}` property flattened at build time; CI produces dynamic versions on `develop` and semantic versions for releases

## Code Instructions

1. First think through the problem, read the codebase for relevant files.
2. Before you make any major changes, check in with me and I will verify the plan.
3. Please every step of the way just give me a high level explanation of what changes you made.
4. Make every task and code change you do as simple as possible. We want to avoid making any massive or complex changes. Every change should impact as little code as possible. Everything is about simplicity.
5. Maintain a documentation file that describes how the architecture of the app works inside and out.
6. Never speculate about code you have not opened. If the user references a specific file, you MUST read the file before answering. Make sure to investigate and read relevant files BEFORE answering questions about the codebase. Never make any claims about code before investigating unless you are certain of the correct answer - give grounded and hallucination-free answers.
7. For implementation use TDD (Test-Driven Development): write or update tests first to define the expected behaviour, verify they fail, then write the minimal implementation to make them pass.
8. Use DRY (Don't Repeat Yourself): extract reusable logic into separate classes, utilities, or components. If the same pattern appears in multiple places, refactor it into a shared helper.

## Directory Structure

```
pom.xml                    # Core of the project — dependency aggregator and build config
logback-test.xml           # Logback console appender config for test execution
LICENSE.txt                # EPL-2.0 full license text
README.md                  # Project overview with dependency architecture diagrams
CONTRIBUTING.md            # Development setup and submission guidelines
CIFLOW.md                  # GitFlow branching strategy and CI/CD workflow documentation
.github/
  workflows/               # 13 GitHub Actions workflows (build, release, merge, etc.)
  ISSUE_TEMPLATE/          # Bug report, feature request, documentation templates
  dependabot.yml           # Dependabot config for daily Maven dependency updates
  labels.yml               # GitHub labels definition
.mvn/
  jvm.config               # JVM memory settings: -Xms1024m -Xmx2048m
  maven-wrapper.properties # Maven 3.8.6 wrapper config
  extensions.xml           # Wagon and profile activation extensions
.vscode/settings.json      # VS Code Java settings (format off, autobuild off)
.zed/settings.json         # Zed editor Java/JDTLS settings
openspec/                  # OpenSpec agentic workflow configuration
```

## Core Modules

This is a single-module project (no `<module>` entries in pom.xml). The value it provides is dependency aggregation:

| Dependency Group | Artifacts | Purpose |
|-----------------|-----------|---------|
| Spring Boot 3.5.0 | `spring-boot-starter`, `spring-boot-starter-data-jdbc` | Core framework and JDBC support |
| JUDO Runtime Core | `judo-runtime-core`, `-spring`, `-dispatcher`, `-accessmanager`, `-accessmanager-api` | Runtime engine, Spring integration, request dispatch, access control |
| JUDO DAO Layer | `judo-runtime-core-dao-core`, `-dao-rdbms`, `-dao-rdbms-hsqldb`, `-dao-rdbms-postgresql` | Data access for HSQLDB and PostgreSQL |
| JUDO Spring DB Support | `judo-runtime-core-spring-hsqldb`, `-spring-postgresql` | Spring-configured database connections |
| PSM Generator SDK | `judo-psm-generator-sdk-core-common`, `-api`, `-impl`, `-spring` | Code generation from PSM models |
| Model Transformation | `judo-tatami-asm2rdbms`, `judo-meta-liquibase.model` | ASM-to-RDBMS transformation, Liquibase model support |
| Utilities | `judo-dao-api`, `judo-sdk-common`, `mapper-api`, `mapper-impl` | DAO contracts, SDK utilities, object mapping |
| Other | `antlr-runtime` 3.2 (Eclipse Xtext requirement), `logback-classic` 1.5.12 (provided scope) | Parser support, logging |

## Technology Stack

### Core Technologies
- **Spring Boot 3.5.0** — application framework
- **JUDO Runtime Core** — domain-driven runtime engine
- **JUDO PSM Generator SDK** — code generation from Platform-Specific Models
- **Lombok 1.18.34** — annotation processing (delombok at `generate-sources` phase)
- **ANTLR 3.2** — parser runtime required by Eclipse Xtext (cannot use >= 3.5)

### Build & Quality
- **Maven 3.8.6** with Maven Wrapper
- **Maven Surefire 3.5.1** — test execution with `--add-opens` JVM args for Java 21 module access
- **JaCoCo 0.8.12** — code coverage (prepare-agent + report)
- **flatten-maven-plugin 1.3.0** — flattens POM for clean distribution
- **sign-maven-plugin 1.1.0** — GPG artifact signing
- **SonarQube** — configured but currently disabled

## Build Commands

```sh
# Full build
mvn clean install

# Run tests only
mvn clean test

# Build with signing and deploy to JUDO Nexus (CI default)
mvn clean deploy -Psign-artifacts -Prelease-judong

# Deploy to Maven Central
mvn clean deploy -Psign-artifacts -Prelease-central

# Deploy to local filesystem (testing)
mvn clean deploy -Prelease-dummy

# Update license headers
mvn process-sources -Pupdate-source-code-license

# Generate PlantUML diagrams for GitHub docs
mvn generate-resources -Pgenerate-github-asciidoc-diagrams
```

### Maven Profiles

| Profile | Purpose |
|---------|---------|
| `sign-artifacts` | GPG-sign all artifacts using `sign-maven-plugin` |
| `release-judong` | Deploy to BlackBelt JUDO Nexus (`nexus.judo.technology`) |
| `release-central` | Deploy to Maven Central via Sonatype OSSRH |
| `release-dummy` | Deploy to local `file:///tmp/` directory for testing |
| `generate-github-asciidoc-diagrams` | Generate PNG diagrams from PlantUML sources |
| `update-source-code-license` | Update EPL-2.0 file headers (excludes `.json` files) |

## Key Configuration Files

| File | Purpose |
|------|---------|
| `pom.xml` | Main POM — dependency aggregation, plugin config, profiles (621 lines) |
| `logback-test.xml` | Console appender with pattern layout for test execution |
| `.mvn/jvm.config` | JVM heap: `-Xms1024m -Xmx2048m`, module `--add-opens` flags |
| `.mvn/maven-wrapper.properties` | Pins Maven to 3.8.6 |
| `.mvn/extensions.xml` | Wagon extensions for artifact transport |
| `.github/dependabot.yml` | Daily Maven dependency updates, ignores guava/slf4j/jruby, max 10 open PRs |
| `.github/labels.yml` | Standard GitHub labels (bug, enhancement, release, etc.) |

## Development Environment

**Required:**
- Java 21 JDK (Zulu distribution recommended)
- Maven 3.8.6+ (or use `./mvnw`)

**Surefire JVM Configuration:**
Tests run with these JVM arguments (configured in pom.xml):
```
--add-opens java.base/java.lang=ALL-UNNAMED
--add-opens java.base/java.util=ALL-UNNAMED
--add-opens java.base/java.time=ALL-UNNAMED
-Dfile.encoding=UTF-8
```

## Git Workflow

- **Main Branch:** `develop` (integration), `master` (releases)
- **Versioning:** `${revision}` property, currently `1.0.4-SNAPSHOT`
- **CI dynamic versions:** `major.minor.qualifier.YYYYMMDD_HHMMSS_commitId_branchName`
- **Release versions:** `major.minor.qualifier` (semantic versioning)
- **Commit convention:** Every commit must reference a JIRA ticket: `JNG-xxx <description>`
- **Branch naming:** `feature/JNG-xxx_summary`, `bugfix/JNG-xxx_summary`, `hotfix/JNG-xxx_summary`, `support/JNG-xxx_summary`
- **CI runner:** Custom `judong` runner with 30-minute timeout
- **Notifications:** Discord webhooks for build status

## Important Notes

1. This project has **no Java source code** — changes are made to `pom.xml` (dependency versions, plugins, profiles) and CI workflows
2. The `antlr-runtime` is pinned to 3.2 because Eclipse Xtext 2.27.0 does not support >= 3.5
3. Logback is declared with `provided` scope — consuming projects supply their own logging implementation
4. The `flatten-maven-plugin` runs at `process-resources` phase, so the published POM differs from the source POM (resolved `${revision}` variable)
5. Dependabot is configured for daily updates but ignores `guava`, `slf4j`, and `jruby-complete` dependencies
6. The CI pipeline automatically creates GitHub releases on `develop` (prereleases) and `master` (final releases)
7. Version bumps are automated: `release.yml` calculates the next version and creates PRs for both `master` and `develop`

## Related Documentation

- [README.md](README.md) — project overview with dependency architecture diagrams
- [CONTRIBUTING.md](CONTRIBUTING.md) — development setup, issue and PR submission guidelines
- [CIFLOW.md](CIFLOW.md) — complete GitFlow branching strategy and GitHub Actions workflow documentation
