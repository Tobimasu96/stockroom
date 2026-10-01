# Stockroom architecture

This is the planned architecture from PROJECT_PLAN.md. Currently implemented: the Spring Boot entry point, embedded Tomcat, Maven Wrapper, and application-context startup test. Business endpoints, security, PostgreSQL, the worker, and delivery infrastructure are planned.

## Application and request flow

```mermaid
flowchart TD
    client["API client: curl, Postman, or optional frontend"]
    subgraph app["One Java / Spring Boot application"]
        tomcat["Embedded Tomcat: accepts HTTP requests"]
        security["Spring Security: sessions, CSRF, role checks"]
        controllers["Controllers: HTTP, DTOs, validation, errors"]
        services["Services: business rules and transactions"]
        repositories["Repositories: database access through JPA"]
        worker["Scheduled worker: claim events, retry, deduplicate"]
        health["Minimal health endpoint; restricted metrics"]
        tomcat --> security
        security --> controllers
        security --> health
        controllers --> services
        services --> repositories
        worker --> repositories
    end
    db[("PostgreSQL: business data, idempotency, outbox, notifications")]
    flyway["Flyway: controlled schema migrations"]
    client -->|"HTTP locally; HTTPS in deployment"| tomcat
    repositories -->|"SQL reads and writes"| db
    flyway --> db
```

Read the main path from top to bottom: Tomcat accepts a request, security checks access, a controller translates the HTTP request, a service applies the business rules, and a repository accesses PostgreSQL. The response returns through the same request path. An HTTPS entry point is required for deployment; its implementation depends on the hosting choice.

The application is organized by feature: `products`, `inventory`, `orders`, and `security`. Each feature has controller/service/repository boundaries where needed. These are packages in one application, sharing one database.

## Stock correctness and background work

```mermaid
flowchart LR
    request["Create order: payload and idempotency key"]
    subgraph tx["One PostgreSQL transaction"]
        idem["Enforce unique key; compare payload"]
        stock["Lock inventory in stable order; validate every line"]
        writes["Write order, price snapshots, stock movements, stock, idempotency result, outbox events"]
        idem --> stock --> writes
    end
    commit["Commit all changes, or roll back all changes"]
    worker["Worker processes committed outbox events"]
    notification["Local notifications: unique event ID prevents duplicates"]
    request --> idem
    writes --> commit
    commit -->|"After successful commit"| worker
    worker --> notification
```

The database enforces constraints as well as application validation. A failed order changes no stock; prices are snapshotted; concurrent retries return one order. Cancellation uses a transactional state transition and restores stock once. The worker may process an event again after a restart, so notification creation must be deduplicated by event ID. Low-stock alerts are suppressed until stock recovers and crosses the threshold again. External email is an optional extension.

## Build, testing, and delivery

```mermaid
flowchart LR
    source["Git repository"] --> checks["GitHub Actions: Maven Wrapper build and tests"]
    postgresTests["Testcontainers: real PostgreSQL tests"] --> checks
    checks -->|"main: build once"| image["Immutable container image in GHCR"]
    image --> staging["Staging: controlled migrations and smoke checks"]
    staging -->|"Release: promote the same tested digest"| production["Production: HTTPS and PostgreSQL"]
```

Pull requests run checks. Main builds and publishes an image, then deploys staging. A release promotes that tested image to production. Deployments are serialized and migrations are controlled; backup, restore, and rollback procedures must be verified. PostgreSQL and the application run locally through Docker Compose when persistence is introduced. Hosting is still undecided.

## Learning checkpoint

Trace a request to create an order through the first diagram. Which component should decide whether enough stock exists, and why should all order lines be handled within one transaction?
