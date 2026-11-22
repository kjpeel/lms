# Dependency Choices

Every dependency is code we have to trust, upgrade and debug. Default to **fewer, smaller, well-maintained** libraries.

## Adding a dependency

Before adding anything, check:

1. Can the stdlib, http4k or an existing dependency already do this?
2. Is it actively maintained (recent releases, issues being answered)?
3. Is the licence permissive (MIT, Apache-2.0, BSD)? No GPL/AGPL without explicit sign-off.
4. How big is it, and how much does it pull in transitively?

Explain the reason for any new dependency in the PR description. Don't add dependencies speculatively.

## API (Kotlin / Gradle)

- Declare all versions in the **Gradle version catalog** (`api/gradle/libs.versions.toml`). No inline version strings in `build.gradle.kts`.
- Import http4k through its BOM (`platform(libs.http4k.bom)`) so all http4k modules stay on the same version.
- Use the Gradle wrapper (`./gradlew`) and commit it.
- Pin exact versions; no dynamic versions (`+`, `latest.release`).

### Approved stack

| Concern | Choice |
|---|---|
| HTTP server/client/routing | http4k (`http4k-core`, `http4k-api-openapi`, `http4k-server-undertow`, `http4k-client-okhttp`) |
| JSON | `http4k-format-moshi` |
| Config | `http4k-config` |
| Result types | Result4k |
| Value types | values4k |
| Database | PostgreSQL, Flyway for migrations, plain JDBC or jOOQ for queries *(choose before first persistence task)* |
| Testing | JUnit 5, Kotest assertions, `http4k-testing-*`, Testcontainers, Konsist |

### Avoid

- **Spring, Ktor, Micronaut, or any other web framework.** http4k is the framework.
- **DI containers** (Koin, Dagger, Guice). Wire by hand.
- **Mocking libraries** (Mockito, MockK). Use fakes.
- **ORMs with runtime magic** (Hibernate/JPA). They hide the SQL and leak into the domain.
- Lombok, reflection-heavy libraries, annotation processors where a plain function would do.

## Web (npm)

- Commit the lockfile. Use one package manager (npm) consistently.
- Pin exact versions (`save-exact=true` in `.npmrc`).
- Put build and test tooling in `devDependencies`.
- Watch the production bundle size when adding UI libraries. Prefer small, tree-shakable packages.

### Approved stack

| Concern | Choice |
|---|---|
| Build | Vite |
| Language | TypeScript (strict) |
| Server state / data fetching | TanStack Query |
| Routing | React Router |
| Forms | React Hook Form + Zod |
| API types | Generated from the API's OpenAPI spec (e.g. `openapi-typescript`) |
| Testing | Vitest, React Testing Library, user-event, MSW, Playwright |
| Lint / format | ESLint, Prettier |

### Avoid

- Global state libraries (Redux etc.) until there's a demonstrated need. Server state belongs in TanStack Query, and local state in React.
- Utility grab-bags (lodash, moment). Use modern JS and `date-fns` / `Intl` when needed.

## Upgrades

- Keep dependencies current with Renovate or Dependabot. Merge small upgrades often instead of large ones rarely.
- Upgrade one library (or one BOM) per PR so failures are easy to bisect.
