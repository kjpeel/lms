---
paths:
  - "api/**"
---

# http4k Patterns

http4k's core idea: **an HTTP app is a function** `(Request) -> Response`. Lean into that. Keep everything as composable functions and avoid magic.

## Application shape

- The whole app is an `HttpHandler` built by a plain function: `fun LmsApp(env: Environment, clock: Clock, courses: CourseRepository, ...): HttpHandler`.
- `Main.kt` is the only place that starts a server: `LmsApp(...).asServer(Undertow(port)).start()`. Everything else is server-agnostic.
- Server-as-a-Function: routes, filters and clients are all `HttpHandler`/`Filter`, so they compose with `.then(...)`.
- No global state, singletons or `object`s holding mutable state.

## Routing

- Use **contract routes** (`contract { ... }` with `OpenApi3`) for the public JSON API, so the OpenAPI spec is generated from code and never drifts.
- One file per resource in `adapters/http` exposing route(s) built from use cases, e.g. `fun CourseRoutes(createCourse: CreateCourse, listCourses: ListCourses): List<ContractRoute>`.
- Route handlers stay thin: extract with lenses, call the use case, map the result to a `Response`. No business logic.

## Lenses

- **All** request input goes through lenses: path, query, headers and body. Never read `request.query("x")` or parse `bodyString()` by hand.
- Define lenses next to the routes that use them, as `val`s.
- Use typed lenses for IDs and values, e.g. `Query.value(CourseId).required("courseId")` with values4k types, so invalid input fails at the edge.
- Identifiers come from **query** lenses, not `Path` lenses. Routes have static paths only (see `api-design.md`).
- Lens failures become `400 Bad Request` through `ServerFilters.CatchLensFailure`.

## JSON

- Use `http4k-format-moshi` (`Body.auto<T>().toLens()`). Don't mix JSON libraries.
- Request and response DTOs are data classes in `adapters/http`, separate from domain types.

## Filters

- Cross-cutting concerns are `Filter`s: error handling, request logging/tracing, auth, CORS.
- Build the filter stack in one place (`App.kt`) in a fixed order: tracing → logging → error catching → auth → routes.
- `DebuggingFilters.PrintRequestAndResponse()` is for local dev only and never ships enabled.

## Error handling

- Use cases return `Result4k`. The HTTP adapter maps `DomainError` subtypes to status codes in one exhaustive `when`.
- Return errors as RFC 9457 `application/problem+json`.
- Don't throw exceptions for expected failures (not found, validation, conflicts). Exceptions are for bugs and infrastructure failure. An outer filter catches them and returns a 500 without leaking internals.

## Configuration

- Use `http4k-config`: declare `EnvironmentKey`s and load with `Environment.ENV overrides Environment.fromResource("local.properties")`.
- Secrets come only from environment variables, never from committed files.
- Fail fast at startup if required config is missing.

## Outbound HTTP

- Driven adapters that call other services take an `HttpHandler` in their constructor, never a concrete client. Wire the real client (e.g. `OkHttp()` / `JavaHttpClient()`) in `Main.kt`.
- Apply `ClientFilters.SetBaseUriFrom(...)` and auth filters at wiring time.
- Every external service gets an **http4k fake** (an `HttpHandler` that simulates it) for tests and local runs. See `testing.md`.

## Observability

- Use http4k `Events` for structured logging; no `println`.
- Propagate trace headers with `ServerFilters.RequestTracing` / `ClientFilters.RequestTracing`.
