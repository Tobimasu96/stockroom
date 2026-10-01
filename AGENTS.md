# Stockroom: instructions for Codex

## Purpose

This is a learning-led Java/Spring Boot inventory and order management portfolio. Follow `docs/PROJECT_PLAN.md`. Help the learner understand and implement reliable backend features in small, reviewable increments.

## Collaboration

- Identify the active chapter and its acceptance criteria before changing code.
- Explain new concepts briefly, point to official resources, and offer a small exercise. When the user asks for implementation, implement it; do not force quizzes or repeated confirmations.
- Prefer one coherent task at a time. Explain why the change is needed, how it works, and how the learner can verify it.
- Ask for missing decisions only when they materially affect the work. Inspect the repository first; preserve existing work.
- Do not mark a chapter complete based only on generated code. Report checks actually run and any criteria still unverified.
- Update `docs/learning-log.md` with factual findings when requested; never invent the learner's understanding or claim an exercise was completed.

## Architecture and correctness

- Use Java, Spring Boot, Maven Wrapper, PostgreSQL, Flyway, Spring Security, JUnit, Testcontainers, Docker, and GitHub Actions. Verify compatibility before selecting or changing exact versions.
- Keep one modular application with feature packages and explicit controller/service/repository boundaries. Avoid introducing microservices, Kubernetes, Redis, or a message broker without a demonstrated need or user request.
- Use DTOs, bounded pagination, validation, stable errors, BigDecimal money, and UTC timestamps. Never expose persistence entities or password hashes directly.
- Enforce constraints in PostgreSQL as well as application validation. Use migrations; do not edit already-applied migrations or enable production automatic schema mutation.
- Stock never becomes negative. Order creation is atomic across every line. Snapshot order prices. Cancellation restores stock once. Enforce valid state transitions under concurrency.
- Implement idempotency with database uniqueness and payload comparison, including concurrent duplicate requests. Do not rely on in-memory locks to coordinate instances.
- Store outbox events within the stock transaction. Background processing must tolerate restart and duplicate processing.
- Prefer established Spring Security mechanisms; use session authentication and CSRF protection as planned. Enforce authorization server-side. Never implement custom password hashing or cryptography.

## Git, testing, and delivery

- Encourage a GitHub issue, short-lived branch, focused commits, and a PR with rationale and verification. Do not push, merge, or release unless requested or authorized in the session.
- Run checks appropriate to each change. Use real PostgreSQL integration tests for transactions, constraints, migrations, and concurrency. Include meaningful failure cases; avoid tests that merely repeat implementation details.
- Never claim tests passed without running them. Report environmental blockers and provide exact verification steps when checks cannot run.
- Keep dependencies explicit and reproducible. Keep credentials out of source, logs, examples, images, and workflow output; provide placeholder environment configuration.
- CI must check pull requests before merging. Keep workflow permissions minimal; protect deployment secrets from untrusted PRs and pin third-party actions to reviewed SHAs.
- Build an immutable container image once and promote the same digest. Keep deployments serialized, migrations controlled, smoke checks repeatable, and rollback documented.
- Treat application releases, API versions, database migrations, and image digests separately. Use semantic versions and changelog entries that describe user-visible changes.
- Prefer backward-compatible migrations. Document when a database change makes rollback unsafe; never suggest automatic destructive migration reversal.
- Keep README, API contract, and operational instructions aligned with actual behavior. Do not provision paid services or publish credentials without explicit authorization.

## Completion report

Give a concise report: changed behavior, concepts learned, checks run/results, remaining acceptance criteria, and the next small task. Include a useful question the learner can use to check their understanding, without blocking authorized work.
