# Stockroom

A learning-led Java/Spring Boot inventory and order management API for a small retailer.

## Current stage

Chapter 1: repository and application setup. This starting skeleton contains the application entry point, a context-loading test, and a GitHub Actions build/test workflow. Business endpoints, persistence, and authentication will follow in small increments.

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

- Confirm the CI failure-demonstration PR passes after the temporary assertion is removed.
- Configure required checks on main where supported, or document the manual merge rule.
- Complete the Git branching/conflict/revert practice in the plan.

## Pull requests and CI

Create a short-lived branch for every change, commit there, and open a PR for `gitdoge523`. Leave the PR unmerged until the colleague approves and the `Build and test` check passes. This is the manual merge rule until required reviews and status checks are configured in GitHub.

`.github/workflows/ci.yml` builds with Java 21 and the Maven Wrapper on every PR and push to `main`. Actions are pinned to commit SHAs; the workflow has read-only repository permissions and no deployment secrets. Inspect the Actions log to see compilation, test results, and packaging. See [GitHub's Maven CI guide](https://docs.github.com/en/actions/tutorials/build-and-test-code/java-with-maven).

## Verification performed

On 1 October 2026, `./mvnw.cmd clean verify` succeeded with Java 21: one context-loading test passed, with no failures or errors, and the executable JAR was built. The packaged application also started on a temporary port and returned the expected 404 for `/`, since no business endpoints exist yet. The smoke-check process was stopped afterward. These checks do not establish fresh-clone or GitHub CI acceptance.

A fresh local Git clone of the initial commit also passed `mvnw.cmd --batch-mode --no-transfer-progress clean verify` on 1 October 2026, with one test and zero failures/errors. This clone used tracked files and the installed JDK/Maven dependency cache; it did not contain the ignored `.tools/` directory. GitHub CI results must be checked separately.

PR #1's GitHub `Build and test` check passed. In [PR #2](https://github.com/Tobimasu96/stockroom/pull/2), a temporary JUnit failure caused [CI run 36865085530](https://github.com/Tobimasu96/stockroom/actions/runs/36865085530) to fail as expected. The next commit removes that assertion; its check must pass before merging. This demonstrates that CI detects a test failure rather than merely compiling successfully.

## First learning exercise

Run `.\mvnw.cmd clean verify` yourself and find `Tests run: 1` and `BUILD SUCCESS` in the output. Open `pom.xml`, `StockroomApplication.java`, and `StockroomApplicationTests.java`: identify where dependencies are declared, where the application starts, and where Spring's startup is tested. The Maven Wrapper selects Maven; the project POM describes what Maven builds. `@SpringBootApplication` enables Spring Boot configuration and component discovery; `@SpringBootTest` checks that the application context can load.

Useful resources: [Spring REST guide](https://spring.io/guides/gs/rest-service/), [Spring Boot system requirements](https://docs.spring.io/spring-boot/system-requirements.html), and [Pro Git](https://git-scm.com/book/en/v2).
