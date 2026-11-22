---
paths:
  - "api/**/*.kt"
  - "api/**/*.kts"
---

# Kotlin Style

## Types and modelling

- **Make illegal states unrepresentable.** Use sealed interfaces for alternatives, and value types for IDs and constrained values (`CourseId`, `EmailAddress`, `PositiveInt`) instead of raw `String`/`Int`.
- Prefer **immutable** data: `val` over `var`, read-only collections, `data class` with `copy()` for changes.
- Validate when a value is constructed (value types with smart constructors) so the rest of the code can trust it.
- Avoid nullable types in the domain where a sealed type says more. Never use `!!`.

## Errors

- Expected failures are values: `Result4k<T, DomainError>`. `DomainError` is a sealed hierarchy per feature.
- Throw only for programmer errors and unrecoverable infrastructure failures.
- Don't catch `Exception` broadly except in the outermost filter.

## Functions and classes

- Prefer small functions and top-level functions over utility `object`s.
- Constructor-inject dependencies. No service locators, no `lateinit` for dependencies.
- Use `internal` to hide adapter details from other modules.
- Name things in the domain's language: `Enrolment`, `Learner`, `Course`, `Lesson`, not `Manager`, `Helper` or `Util`.

## Formatting

- Run ktlint (via Gradle plugin) and fix all findings before committing.
- Use trailing commas in multi-line parameter lists.
- Use explicit return types on public functions.
