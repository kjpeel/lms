---
paths:
  - "api/**"
---

# Architecture: Hexagonal (Ports & Adapters)

The API is built as a hexagon. Business logic sits in the middle and knows nothing about HTTP, databases or third-party services. Everything outside talks to it through ports.

## Layers and dependency direction

```
adapters  ──►  application  ──►  domain
(http4k, db,     (use cases)      (model, rules, ports)
 clients)
```

Dependencies point **inward only**. Inner layers never import from outer layers.

| Layer | Contains | May depend on |
|---|---|---|
| `domain` | Entities, value objects, domain errors, driven port interfaces (e.g. `CourseRepository`, `Clock`) | Kotlin stdlib, Result4k, values4k. **Nothing else.** |
| `application` | Use cases (driving ports), one per business action (e.g. `EnrolStudent`) | `domain` |
| `adapters` | Driving adapters (http4k routes) and driven adapters (Postgres repos, HTTP clients, email) | `application`, `domain`, frameworks |
| root (`App.kt`, `Main.kt`) | Wiring: build adapters, pass them into use cases, compose the `HttpHandler` | everything |

## Package layout

Feature-first inside each layer:

```
api/src/main/kotlin/lms/
  domain/
    course/        Course.kt, CourseId.kt, CourseRepository.kt
    enrolment/     ...
  application/
    course/        CreateCourse.kt, ListCourses.kt
  adapters/
    http/          CourseRoutes.kt, lenses, DTOs, error mapping
    persistence/   PostgresCourseRepository.kt
    clients/       outbound HTTP integrations
  App.kt           fun LmsApp(deps): HttpHandler
  Main.kt          reads Environment, starts the server
```

## Rules

- **No framework types in `domain` or `application`.** No `Request`, `Response`, lenses, JDBC, JSON annotations. If you need to import `org.http4k.core` in there, the code belongs in an adapter.
- **Ports are owned by the domain.** Port interfaces describe what the domain *needs* in its own language (`findById(CourseId)`), not what the technology offers (`executeQuery(sql)`).
- **DTOs stay in adapters.** Map between DTOs and domain types at the edge. Never serialise domain objects directly to JSON.
- **Use cases are small and explicit.** Model each use case as a class or function with a single `invoke`. It takes its ports via the constructor and returns a `Result4k` (`Result<Success, DomainError>`).
- **No DI container.** Wire dependencies by hand in `App.kt` and `Main.kt`. Constructor arguments are the dependency graph.
- **Side effects go through ports.** Time (`Clock`), ID generation, randomness, I/O. This keeps the core deterministic and testable.
- **Enforce the boundaries with a test.** Use a [Konsist](https://docs.konsist.lemonappdev.com/) architecture test that fails the build if `domain` or `application` imports from `adapters` or `org.http4k`.

## When in doubt

Ask: "Could I swap this for an in-memory fake without changing the domain?" If not, a port is missing.
