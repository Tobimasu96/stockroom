# Stockroom

A learning-led Java/Spring Boot inventory and order management API for a small retailer.

## Current stage

Chapter 1: repository and application setup. This starting skeleton contains the application entry point and a context-loading test. Business endpoints, persistence, authentication, and CI will follow in small increments.

See [the project plan](docs/PROJECT_PLAN.md) for deliverables and acceptance criteria, and [AGENTS.md](AGENTS.md) for collaboration rules.

See [the architecture diagrams](docs/architecture.md) for the planned request flow, stock transactions, background processing, and delivery pipeline. The document identifies what is implemented today and what is planned.

## Prerequisites

- JDK 21, with `JAVA_HOME` pointing to the JDK directory and Java on `PATH`.
- Git on `PATH`.
- Internet access for the first Maven Wrapper run to download Maven and dependencies.

Versions: Java 21, Spring Boot 4.1.1, Maven 3.9.16, Maven Wrapper 3.3.4. A separate Maven installation is unnecessary. PostgreSQL and Docker are introduced in Chapter 3.

## Build and run

From the repository root in PowerShell:

```powershell
java -version
.\mvnw.cmd clean verify
.\mvnw.cmd spring-boot:run
```

On Linux/macOS, use `sh ./mvnw clean verify` and `sh ./mvnw spring-boot:run`.

The server starts on port 8080. There are no API endpoints yet; a request to `/` returns 404. Stop the server with Ctrl+C.

## Structure

```text
.mvn/wrapper/           Pinned Maven distribution configuration
docs/PROJECT_PLAN.md   Chapters and acceptance criteria
src/main/java/         Application code under com.stockroom
src/main/resources/    Application configuration
src/test/java/         Automated tests
pom.xml                Dependencies and build configuration
mvnw, mvnw.cmd         Maven Wrapper scripts
```

Feature packages for products, inventory, orders, and security will be added as their features are implemented, with controller/service/repository boundaries.

## First commit

Review the files and run the build. Then:

```powershell
git init -b main
git add .
git diff --cached --stat
git diff --cached
git commit -m "chore: create Stockroom starting structure"
```

If Git requests an identity, configure your name and email for this repository. Nothing needs to be pushed yet. Build outputs, IDE metadata, and local secrets are ignored; inspect staged files before committing.

## Chapter 1 verification still required

- Run `clean verify` from a fresh clone.
- Add GitHub Actions for pull requests and main; demonstrate a failing test fails CI and its fix passes.
- Configure required checks on main where supported, or document the manual merge rule.
- Complete the Git branching/conflict/revert practice in the plan.

## Verification performed

On 1 October 2026, `./mvnw.cmd clean verify` succeeded with Java 21: one context-loading test passed, with no failures or errors, and the executable JAR was built. The packaged application also started on a temporary port and returned the expected 404 for `/`, since no business endpoints exist yet. The smoke-check process was stopped afterward. These checks do not establish fresh-clone or GitHub CI acceptance.

## First learning exercise

Run `.\mvnw.cmd clean verify` yourself and find `Tests run: 1` and `BUILD SUCCESS` in the output. Open `pom.xml`, `StockroomApplication.java`, and `StockroomApplicationTests.java`: identify where dependencies are declared, where the application starts, and where Spring's startup is tested. The Maven Wrapper selects Maven; the project POM describes what Maven builds. `@SpringBootApplication` enables Spring Boot configuration and component discovery; `@SpringBootTest` checks that the application context can load.

Useful resources: [Spring REST guide](https://spring.io/guides/gs/rest-service/), [Spring Boot system requirements](https://docs.spring.io/spring-boot/system-requirements.html), and [Pro Git](https://git-scm.com/book/en/v2).
