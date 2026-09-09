# judo-jsl-springboot-starter — module agent doctrine

## Module purpose

This repository publishes `hu.blackbelt.judo:judo-spring-boot-starter`, the single dependency
a **generated JSL application** declares to become a runnable JUDO Spring Boot service. Put
it on the classpath and the application inherits, via the transitive
`judo-runtime-core-spring` configuration classes, a fully wired JUDO runtime it never writes
itself: `JudoModelLoaderConfiguration` locates and loads the compiled model (by `modelName`,
or by searching when unset) and exposes a `JudoModelLoader`; `JudoBaseServiceConfiguration`
contributes the cross-cutting infrastructure beans (`ExtendableCoercer`, `DataTypeManager`,
`IdentifierProvider`, `MetricsCollector`, `Context`, `AuthenticationInterceptorProvider`,
`OperationCallInterceptorProvider`, `DispatcherFunctionProvider`, `PasswordPolicy`,
`Export`); and `JudoDefaultSpringConfiguration` builds the execution stack — `DAO`,
`QueryFactory`, `RdbmsBuilder`, `RdbmsResolver`, `Select`/`ModifyStatementExecutor`,
`InstanceCollector`, `Dispatcher`, `AccessManager`, `ActorResolver`, `PayloadValidator`,
`ValidatorProvider`, `IdentifierSigner`, `VariableResolver`, `TransformationTraceService`.
Database-specific beans arrive from the two dialect configurations also pulled in
(`JudoHsqldbSpringConfiguration`, `JudoPostgresqlSpringConfiguration`: dialect, `Sequence`,
`RdbmsParameterMapper`, `MapperFactory`), so the application picks HSQLDB or PostgreSQL by
datasource, not by editing dependencies. The starter also drags in the PSM generator SDK
(`judo-psm-generator-sdk-core-*`) so the same project can run code generation from its model.

The starter itself declares **no `@AutoConfiguration` and no Java source** — its entire value
is curation: one `pom.xml` that pins a mutually consistent set of Spring Boot 3.5.0, JUDO
runtime, DAO, dispatcher, access-manager and generator versions, so a generated project never
resolves a mismatched combination. Editing this repository means editing that POM, the CI
workflows, or nothing.

**Repository:** BlackBeltTechnology/judo-jsl-springboot-starter
**Artifact:** `hu.blackbelt.judo:judo-spring-boot-starter`, packaging `jar`, version `${revision}` (currently `1.0.4-SNAPSHOT`)
**License:** Eclipse Public License 2.0 (EPL-2.0)
**Java Version:** 21 (Zulu distribution in CI)
**Build System:** Maven 3.8.6 with Maven Wrapper (`./mvnw`)

## Reactor map

`pom.xml` declares **no `<modules>`** — nothing to descend into. This is a single-module,
non-aggregator reactor emitting one jar. `src/main/resources` holds only `placeholder`
("Placeholder file for JAR creation"), which exists so the jar is not empty; there is no
compiled code.

What the single POM contributes instead of submodules — the curated dependency aggregation:

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

Two BOM imports govern the versions: `spring-boot-dependencies` and
`judo-runtime-core-dependencies`. Bump either and every group above moves together.

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
src/main/resources/
  placeholder              # Empty-jar placeholder; no compiled sources in this repository
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

## Build Inputs Outside Any Source Directory

These govern the build but live in directories that carry no `AGENTS.md`; their behaviour is
recorded here as architecture, not as a per-file index.

- **`pom.xml`** — the whole project: dependency aggregation, plugin config, profiles (621
  lines). Every change to this repository is a change to this file.
- **`logback-test.xml`** — console appender with pattern layout for test execution.
- **`.mvn/jvm.config`** — JVM heap `-Xms1024m -Xmx2048m` plus module `--add-opens` flags.
- **`.mvn/maven-wrapper.properties`** — pins Maven to 3.8.6.
- **`.mvn/extensions.xml`** — wagon extensions for artifact transport; deploy profiles
  depend on these resolving.
- **`.github/dependabot.yml`** — daily Maven dependency updates; ignores guava/slf4j/jruby,
  max 10 open PRs.
- **`.github/labels.yml`** — standard GitHub labels (bug, enhancement, release, etc.).

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

<!-- dox-doctrine -->
## Documentation Update Protocol (WRITE discipline)

Per-directory `AGENTS.md` files form a tree. Each directory `AGENTS.md` is the
per-file record for the files in that directory. This module-root `AGENTS.md`
holds doctrine + architecture pointers only — never a per-file index.

**Keep the root lean.** This file loads into every agent turn — every byte costs
tokens on every turn. A verbose root file buries the rules the model must follow
(signal dilution) and measurably degrades adherence; a lean file keeps doctrine
salient. Default assumption: your update does NOT belong in the root — route it
by the table below.

**Route every doc update by kind:**

| Kind of update | Goes in |
|---|---|
| New file in a directory, or its per-file detail / change history | Nearest directory `AGENTS.md`. Add a `` | `<basename>` | <purpose> | `` row, path-alphabetical. |
| Data flow, protocol, architecture rationale | `docs/architecture.md` or a `docs/<topic>.md` |
| End-user / developer setup | `README.md` |
| Cross-cutting rule every agent needs every turn (rare) | this module-root `AGENTS.md` |

**Read before editing (chain walk).** Before editing a file, read the nearest
`AGENTS.md` chain root→leaf so you know the file's recorded purpose, contracts,
and change history. Do not edit blind.

**Update after editing (closeout pass).** After changing a file, update its row
in the nearest directory `AGENTS.md`: find the file's row, update its purpose in
place; if absent, add it in path-alphabetical order. New directory → scaffold
its `AGENTS.md`. One row per file. The purpose carries a one-line summary, key
exported symbols, contracts/invariants, and `See change: <id>` history.

**Row style (caveman).** Short declarative fragments. Drop articles. Subject →
verb → object, present tense. One fact per row. Prefer concrete tokens (paths,
symbols, env vars) over prose. Keep identifiers verbatim.

**Size rule — split an over-large directory `AGENTS.md` file-based.** pi
auto-injects a directory `AGENTS.md` on every turn when cwd sits at/below it, so
an over-large directory `AGENTS.md` is not supported. Split it file-based: a row
exceeding the length threshold promotes to a per-file `<File>.AGENTS.md`
sidecar carrying that file's full detail (including every `See change:`). The
sidecar is pull-only — its name is not `AGENTS.md`, so pi never auto-injects it
— yet it stays search-indexed (`agents` doc_type). The directory `AGENTS.md`
keeps a one-line summary plus a `→ see `<File>.AGENTS.md`` pointer. Rows within
the threshold stay verbatim (lossless).

## Finding docs (READ discipline)

`kb_*` tools are faster and cheaper than raw search — they return a one-line
purpose + key exports per file, not raw bytes. **This fires on the ACTION, not
the intent** — before you `grep`/`rg` for a symbol, `cat`/read a file to learn
what it does, or chase an import, the kb call goes first. It fires **even
mid-task when you already know the file**; knowing the file does not exempt you.
When your reflex is the left column, run the right column instead:

| You're about to… | Do this FIRST instead |
|---|---|
| `grep -rn "SymbolName" src/` — find where a fn / type / const lives | `kb_search --doc-type agents "SymbolName"` — tree indexes key exports per file |
| `grep -rn "feature\|topic" src/` — how does X work / where's X handled | `kb_search "feature topic"` |
| `cat` / read a file just to learn its purpose before editing | `kb agents <path>` — one-line purpose + exports + change history |
| chase imports / callers across files | `kb_neighbors <path\|heading>` |
| read one doc section in full | `kb_get <path> <section>` |

**Fall-through (explicit):** if the kb call returns nothing relevant, `rg` /
source read is allowed — then add the missing directory `AGENTS.md` row per the
WRITE discipline. kb does NOT replace grep; it goes first.
