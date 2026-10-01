# Stockroom: inventory and order management API

## Goal and stack

Build a deployable backend for a small retailer: manage products, receive stock, create orders, cancel orders, and report low stock. The portfolio should prove that you can keep business data correct, secure an API, and ship changes safely.

Use **Java + Spring Boot + Maven + PostgreSQL**, with Flyway migrations, Spring Security, JUnit, Testcontainers, Docker Compose, GitHub Actions, and GitHub Container Registry (GHCR). Java builds on your existing knowledge and Spring Boot has current business-backend demand; this is a practical choice, not a claim that one language dominates every job market. Use Java 21 LTS and a compatible, supported stable Spring Boot release from Spring Initializr. Record exact versions at setup; avoid milestone/snapshot releases.

Start as one application with packages by feature: products, inventory, orders, security. Use controllers for HTTP, services for business rules, and repositories for persistence. A frontend is optional after the backend is finished.

**Planning assumption:** 8–10 hours/week, roughly 12–16 weeks. Work by acceptance criteria, not calendar deadlines. Chapters 1–4 are foundations, 5–7 deliver the business features, and 8–10 make the project production-minded and portfolio-ready.

## Scope and rules

- One retailer, one warehouse, one currency; integer stock quantities and BigDecimal prices. No real payments or customer personal data.
- A product has a unique SKU, name, price, active flag, and low-stock threshold. Archive products instead of deleting referenced records.
- Stock changes have a quantity, reason, actor, and timestamp. Stock cannot become negative.
- Creating an order immediately deducts stock. Either all order lines succeed or none do. Snapshot prices on order lines; later price changes must not rewrite history.
- Orders move from PLACED to FULFILLED or CANCELLED. Cancellation restores stock exactly once; fulfilled orders cannot be cancelled.
- Repeating an order request with the same idempotency key and payload returns the original result. Reusing the key with a different payload returns a conflict.
- ADMIN manages accounts/products; STAFF receives stock and handles orders; VIEWER reads reports. Public endpoints expose only minimal health information.
- Low-stock alerts are stored locally first. External email, multiple warehouses, payments, and microservices are optional extensions.

## How to work through each chapter

1. Read the listed resource sections until you understand the named concepts; you do not need to finish whole manuals.
2. Complete the small exercise independently before implementing the feature.
3. Open a GitHub issue with the deliverable and acceptance criteria; implement it on a short-lived branch.
4. Ask Codex for explanations, small changes, and review. Read every diff and run the checks yourself.
5. Open a pull request, write what changed and how you verified it, then merge after CI passes. Explain the chapter's checkpoint without relying on generated notes.

Keep `docs/learning-log.md`: concept learned, bug encountered, cause, fix, and one decision you can defend. A solo PR is useful practice; it does not provide independent review.

## 1. Set up the repository and Git workflow — 1 week

**Deliver:** GitHub repository, Spring Boot skeleton, Maven Wrapper, `.gitignore`, README with prerequisites/run instructions, first test, and a CI workflow that builds/tests on pull requests and main. Copy the accompanying `AGENTS.md` into the repository root and this plan into `docs/PROJECT_PLAN.md`.

**Learn:** Java records, collections, exceptions, BigDecimal, interfaces, dependency injection; Maven dependencies and lifecycle; Git working tree/index/commits, branches, remotes, push/pull, and PRs. Learn why build outputs and secrets are excluded.

**Read:** [Java learning](https://dev.java/learn/) — collections, exceptions, classes; [Pro Git](https://git-scm.com/book/en/v2) — chapters 1–3; [Spring REST guide](https://spring.io/guides/gs/rest-service/); [GitHub Java CI](https://docs.github.com/en/actions/tutorials/build-and-test-code/java-with-maven).

**Exercise:** Make two branches edit the same practice file, cause a merge conflict, resolve it, and explain the resolution. Use `git revert` to undo a pushed practice commit; compare this with resetting local unshared work.

**Done when:** A fresh clone builds using the wrapper; an intentionally failing test makes CI fail; fixing it makes CI pass. Configure required checks on main if your repository/account supports them; otherwise document the manual merge rule.

**Checkpoint:** Explain what a commit contains, what a PR does, and why a passing local build may differ from CI.

## 2. Design the domain and HTTP contract — 1 week

**Deliver:** Entity relationship diagram, business rules, API examples, and an OpenAPI specification for `/api/v1/products`, stock adjustments, orders, cancellation, and low-stock reports. Record the architecture choice in `docs/decisions/001-architecture.md`.

**Learn:** HTTP methods/status codes, JSON, REST resource design, validation, request/response DTOs, pagination, stable error formats, primary/foreign keys, uniqueness, and normalization. Distinguish an API contract from the database schema.

**Read:** [MDN HTTP overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview); [OpenAPI specification](https://spec.openapis.org/oas/latest.html); [PostgreSQL tutorial](https://www.postgresql.org/docs/current/tutorial.html) — tables, queries, joins.

**Exercise:** Write sample requests and responses for a valid order, missing product, invalid quantity, insufficient stock, and conflicting idempotency key. Draw how products, inventory, stock movements, orders, and order lines relate.

**Done when:** Each endpoint has inputs, outputs, permissions, and failure responses; order transitions and stock invariants are unambiguous.

**Checkpoint:** Explain when you would use 400, 401, 403, 404, and 409.

## 3. Build persistence and product management — 1–2 weeks

**Deliver:** Local PostgreSQL through Docker Compose; Flyway migrations; product create/read/update/archive endpoints; filtering and bounded pagination; validation and consistent errors.

**Learn:** SQL SELECT/INSERT/UPDATE/JOIN, database constraints, indexes, JPA mappings, repository queries, transactions, DTO mapping, migrations, and environment-based configuration. Use SQL directly before relying on ORM abstractions. Disable automatic production schema mutation.

**Read:** [Spring JPA guide](https://spring.io/guides/gs/accessing-data-jpa/); [PostgreSQL constraints](https://www.postgresql.org/docs/current/ddl-constraints.html); [Flyway documentation](https://documentation.red-gate.com/flyway) — versioned migrations and validation; [Docker Compose](https://docs.docker.com/compose/).

**Exercise:** Write a SQL join and a paginated query yourself. Apply two migrations to an empty database, then to an existing database; observe why editing an applied migration breaks validation.

**Done when:** Data survives restart, duplicate SKUs are rejected, invalid prices fail, pagination has a maximum size, and migrations work on a fresh database. Include PostgreSQL-backed repository tests.

**Checkpoint:** Explain what the ORM does and what the database must still enforce.

## 4. Add authentication and permissions — 1–2 weeks

**Deliver:** Spring Security login/logout with server-side sessions, password hashing, role checks, and protected business endpoints. Provide a local account setup procedure; no embedded passwords. Document using the API with a cookie jar and CSRF token.

**Learn:** Authentication versus authorization, password hashing versus encryption, session cookies, CSRF, CORS, HTTPS, secret handling, and least privilege. Keep CSRF protection for cookie-authenticated writes. Prefer Spring's established mechanisms over custom cryptography or a homemade JWT system.

**Read:** [Spring Security reference](https://docs.spring.io/spring-security/reference/) — servlet authentication, authorization, password storage, CSRF; [OWASP API risks](https://owasp.org/www-project-api-security/).

**Exercise:** Write a permission matrix. Try each protected operation as anonymous, VIEWER, STAFF, and ADMIN; predict the response before sending it.

**Done when:** Tests prove unauthorized reads/writes fail, restricted operations return 403 for insufficient roles, password hashes never appear in responses, and session-authenticated writes enforce CSRF.

**Checkpoint:** Explain why hiding a button cannot enforce permission and why CORS is not authentication.

## 5. Receive stock and create orders safely — 2 weeks

**Deliver:** Stock adjustment history and transactional order creation with price snapshots, plus database-backed idempotency. Duplicate product lines are combined or rejected consistently. Reject nonpositive quantities and adjustment results below zero.

**Learn:** ACID, transaction boundaries, isolation, race conditions, row locks, deadlocks, unique constraints, atomic updates, and idempotency. Lock inventory rows in stable product-ID order, validate all quantities, and commit stock changes, movements, order, and idempotency record together. Enforce an idempotency uniqueness constraint; concurrent repeats must return one result, not two orders.

**Read:** [Spring transaction guide](https://spring.io/guides/gs/managing-transactions/); [PostgreSQL concurrency](https://www.postgresql.org/docs/current/mvcc.html) — isolation and explicit locking; [Spring transaction reference](https://docs.spring.io/spring-framework/reference/data-access/transaction.html).

**Exercise:** Reproduce a lost update with two concurrent database sessions, then prevent it with a lock or conditional update. Explain why a Java `synchronized` block does not coordinate two application instances.

**Done when:** With one unit available, two concurrent orders produce exactly one success; a failed multi-line order changes no stock; identical concurrent retries create one order; a changed payload with the same key fails. Verify using real PostgreSQL.

**Checkpoint:** Trace what happens if the client loses its connection after the database commits.

## 6. Fulfil/cancel orders and produce reports — 1 week

**Deliver:** Validated order transitions, exactly-once stock restoration on cancellation, order filtering, and a low-stock report. Keep state changes and their audit history transactional.

**Learn:** State machines, conditional updates/locking, reporting joins/aggregations, index selection, query plans, and the N+1 query problem.

**Read:** [PostgreSQL EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html); [Spring Data JPA](https://docs.spring.io/spring-data/jpa/reference/) — query methods and fetching.

**Exercise:** Seed realistic data; inspect a slow query plan, add a justified index, and compare. Run cancellation twice and concurrently with fulfilment.

**Done when:** Repeated cancellation restores stock once; concurrent fulfilment/cancellation produces one valid final state; historical prices remain unchanged; report queries are bounded and indexed where useful.

**Checkpoint:** Explain why checking order status before a transaction is insufficient.

## 7. Add reliable background processing — 1 week

**Deliver:** A scheduled database-backed worker processes low-stock events into local notification records. Store events in an outbox in the same transaction as the stock change. Track pending/processed/failed states and retries. Deduplicate alerts by event ID; suppress repeated alerts until stock recovers and crosses the threshold again.

**Learn:** Transactional outbox, at-least-once processing, duplicate handling, bounded retries/backoff, worker claiming, and failure recovery. Begin with one worker; document how row claiming would support multiple workers later.

**Read:** [Spring scheduling guide](https://spring.io/guides/gs/scheduling-tasks/); [AWS transactional outbox pattern](https://docs.aws.amazon.com/prescriptive-guidance/latest/cloud-design-patterns/transactional-outbox.html).

**Exercise:** Stop the worker after it writes a notification but before it marks the event processed. Restart and prove deduplication prevents another notification.

**Done when:** Failed work survives restart, retries are bounded, poison events are inspectable, and duplicate delivery has no duplicate local effect. External email remains an extension because its delivery guarantees need separate treatment.

**Checkpoint:** Explain why sending an email inside a database transaction cannot make both systems atomic.

## 8. Strengthen tests and observability — 1–2 weeks

**Deliver:** Unit tests for business rules, HTTP/security tests, PostgreSQL integration tests with Testcontainers, structured logs with request IDs, minimal public health checks, restricted metrics, and a repeatable load-test script. Tests have been added throughout; this chapter fills gaps.

**Learn:** Test doubles versus real dependencies, deterministic fixtures, integration testing, negative cases, latency percentiles, throughput, connection pools, logs versus metrics, and diagnosing bottlenecks. Never log credentials or session cookies.

**Read:** [Spring web testing](https://spring.io/guides/gs/testing-web/); [Testcontainers Java](https://java.testcontainers.org/); [Spring Actuator](https://docs.spring.io/spring-boot/reference/actuator/); [k6 documentation](https://grafana.com/docs/k6/latest/).

**Exercise:** Introduce a broken stock update and prove a test catches it. Load-test a seeded dataset and use a request ID to investigate a deliberately triggered failure.

**Done when:** Critical concurrency, rollback, permissions, retries, and migration scenarios run in CI; publish dataset size, hardware, concurrency, error rate, and p95 latency. Establish your own measured baseline instead of claiming production-scale performance.

**Checkpoint:** Explain why high coverage alone does not establish correctness.

## 9. Build GitHub CI/CD and deploy — 1–2 weeks

**Deliver:** A multi-stage Dockerfile running as a non-root user; CI for build, formatting/static analysis, tests, migration validation, and dependency scanning; image publishing to GHCR; deployment to staging and production through GitHub Actions. Use one chosen container host with PostgreSQL and HTTPS; document its setup and costs before provisioning.

**Learn:** CI versus continuous delivery/deployment, workflow triggers, jobs/artifacts/cache, minimum token permissions, secrets/environments, image tags versus immutable digests, health checks, deployment concurrency, migrations, backups, and rollback.

**Read:** [GitHub Java CI](https://docs.github.com/en/actions/tutorials/build-and-test-code/java-with-maven); [Publishing Docker images](https://docs.github.com/en/actions/tutorials/publish-packages/publish-docker-images); [GitHub deployment environments](https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments); [Docker build practices](https://docs.docker.com/build/building/best-practices/).

**Pipeline:** PR → checks only; main → checks, build immutable image, publish, deploy staging, smoke-test; release tag → promote that same tested image to production. Serialize deployments. Run migrations as a controlled release step, with deployment stopped on failure. Keep production approval if your GitHub plan supports it; otherwise use a documented manual workflow gate. Do not expose deployment credentials to fork PRs. Pin third-party actions to reviewed commit SHAs.

**Exercise:** Deploy a faulty release in staging and restore the prior image. Back up PostgreSQL and restore into a separate database. Perform an additive migration while the previous application version still runs.

**Done when:** A fresh runner can build; staging deploys through Actions; production release is traceable to source SHA and image digest; smoke checks pass; rollback and restore instructions are tested. Never assume reverting an image undoes a destructive migration.

**Checkpoint:** Explain which changes make application rollback unsafe and how expand/contract migrations help.

## 10. Release and present the portfolio — 1 week

**Deliver:** Release `v1.0.0`, changelog, README, API examples, architecture diagram, demo dataset, short demo recording, and operations runbook covering configuration, deployment, rollback, backup/restore, and common failures.

**Learn:** Semantic versioning, annotated Git tags, GitHub releases, backward compatibility, API versioning, migration sequencing, dependency lock/pinning practices, and communicating tradeoffs. Product release versions, `/api/v1` contract versions, migration numbers, and image digests describe different things.

**Read:** [Semantic Versioning](https://semver.org/); [Git tagging](https://git-scm.com/book/en/v2/Git-Basics-Tagging); [GitHub releases](https://docs.github.com/en/repositories/releasing-projects-on-github/about-releases).

**Exercise:** Ship a backward-compatible field in `v1.1.0` and a bug fix in `v1.1.1`. Write a proposal for a breaking change, with deprecation and migration steps; do not implement a second API version just to demonstrate a label.

**Done when:** Someone can clone/run the project from the README and reproduce the race-condition test. The demo shows receiving stock, successful/rejected orders, safe retries, cancellation, and a release through CI/CD. Record limitations honestly.

**Checkpoint:** Give a five-minute walkthrough: problem, schema, transaction boundary, hardest bug, verification, deployment, and one tradeoff.

## Portfolio completion checklist

- [ ] Runnable API with example requests and clear setup.
- [ ] Correct stock/order behavior under concurrency and retries.
- [ ] Authentication and tested role permissions.
- [ ] Versioned database migrations and reproducible environment.
- [ ] Meaningful tests passing in GitHub Actions.
- [ ] Demonstrated staging/production delivery, rollback, and database restore.
- [ ] Git issues/PRs/tags/releases that tell a clear engineering story.
- [ ] Learning log and a demo you can explain without Codex.

## Starting prompt for Codex

> Read AGENTS.md and docs/PROJECT_PLAN.md. Help me complete chapter 1 in small increments. First explain its concepts and propose one practice exercise. Inspect the repository, then suggest the smallest useful implementation task. Let me attempt it before providing a full solution unless I ask you to implement it. Keep acceptance criteria visible and explain how I can verify the result.

## Market context

Selection researched on 1 October 2026. [IT Jobs Watch Spring Boot UK trends](https://www.itjobswatch.co.uk/jobs/uk/spring%20boot.do) provides a current job-advertisement signal, with regional and sampling limitations. [Stack Overflow 2025 technology survey](https://survey.stackoverflow.co/2025/technology?frame=0) supports the relevance of PostgreSQL and Docker; adoption surveys are not hiring guarantees. The plan prioritizes transferable backend skills over tool popularity.
