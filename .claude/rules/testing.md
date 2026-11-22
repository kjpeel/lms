# Testing

Tests describe behaviour and give us confidence to change things. The hexagonal design exists partly so that most tests are fast and in-memory.

## General rules

- **Write the test first** where practical. Every bug fix starts with a failing test that reproduces it.
- A task isn't done until its tests pass locally: `./gradlew test` in `api/`, `npm test` in `web/`.
- Tests are deterministic: inject `Clock`, ID generators and randomness. No `Thread.sleep`, no reliance on wall-clock time or test order.
- One behaviour per test. Arrange / Act / Assert, separated by blank lines.
- Name tests as sentences: ``fun `rejects enrolment when course is full`()``.
- **No mocking libraries** (no Mockito, no MockK). Use hand-written fakes that implement the port. Fakes are real code: keep them simple and correct, and verify them with contract tests.
- Don't test private functions or implementation details. Test through the public API of the unit.

## API (Kotlin) test pyramid

| Level | What | How | Speed |
|---|---|---|---|
| Domain / unit | Entities, value objects, use cases | Plain JUnit, in-memory fakes for ports | ms |
| HTTP (in-memory) | Routes, lenses, error mapping, full filter stack | Call `LmsApp(fakes)` directly as a function: `app(Request(GET, "/courses"))`. No server, no network. | ms |
| Port contract | Each driven adapter meets its port's contract | Abstract contract test class, run against the fake **and** the real adapter | fast / slow |
| Integration | Real Postgres, real HTTP server | Testcontainers; `LmsApp(...).asServer(...)` on a random port | seconds |
| Smoke | Deployed app is alive | Handful of checks against a running environment | - |

Most tests should sit in the first two rows.

### Patterns

- **Contract tests for ports.** For each port (e.g. `CourseRepository`), write `abstract class CourseRepositoryContract { abstract val repository: CourseRepository }` with the behavioural tests. Then extend it with `InMemoryCourseRepositoryTest` and `PostgresCourseRepositoryTest`. This keeps the fake honest.
- **Same tests, in-memory or over the wire.** HTTP acceptance tests take an `abstract val http: HttpHandler` so the same suite can run against the in-memory app or a real server.
- **http4k fakes for external services.** Each outbound integration has a fake `HttpHandler` in `src/testFixtures` that the client adapter's contract test also runs against.
- **Approval tests** (`http4k-testing-approval`) for JSON response bodies and the generated OpenAPI spec. Review approved files in PRs like code.
- **Architecture test** (Konsist) enforces the layer rules in `architecture.md`.

### Tooling

- JUnit 5 as the runner.
- `http4k-testing-kotest` / Kotest assertions for matchers (`response shouldHaveStatus OK`).
- Testcontainers for Postgres and anything else with real infrastructure.
- Shared fakes and builders live in the Gradle `testFixtures` source set.

## Web (React) testing

- **Vitest** with **React Testing Library** for components and hooks.
- Query the way a user would: `getByRole`, `getByLabelText`. Avoid `getByTestId` unless there's no accessible alternative.
- Use **MSW** (Mock Service Worker) to fake the API at the network level. Don't mock `fetch` or the API client module.
- Use `@testing-library/user-event` for interactions, not `fireEvent`.
- **Playwright** for a small number of end-to-end journeys across web and API (e.g. sign in, enrol, complete a lesson). Keep it to critical paths only.
- No snapshot tests of component markup. They break on every change and catch little.

## Coverage

Coverage is a signal, not a target. Don't write tests just to raise a number. Do make sure every use case and every error branch in the HTTP mapping is exercised.
