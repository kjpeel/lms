---
paths:
  - "web/**"
---

# Frontend (React + TypeScript)

The same principle as the API applies: keep logic out of the framework edges. Components render. Hooks and plain modules hold behaviour.

## Structure

```
web/src/
  api/           generated types + thin typed client (fetch wrapper)
  features/
    courses/     components, hooks, routes, tests for one feature
    enrolment/
  components/    shared, presentational UI building blocks
  lib/           framework-free helpers (formatting, validation)
  app/           router, providers, layout
```

- Organise by feature first. Code moves to `components/` or `lib/` only once a second feature needs it.
- Colocate tests: `CourseList.tsx` sits next to `CourseList.test.tsx`.

## TypeScript

- `strict: true`, plus `noUncheckedIndexedAccess`. No `any`; use `unknown` and narrow.
- Don't use `as` casts to silence the compiler. Fix the types.
- API types come from the generated OpenAPI types. Validate untrusted data (forms, URL params) with Zod.

## React

- Use function components and hooks only.
- **Server state lives in TanStack Query.** Don't copy fetched data into `useState`/context.
- Keep components small. Extract a custom hook when a component mixes data fetching, state logic and markup.
- Keep business rules out of components. Put them in `lib/` or feature modules as pure functions that are easy to unit test.
- Handle loading, error and empty states explicitly for every query.

## Accessibility

- Use semantic HTML first (`button`, `nav`, `label`), and ARIA only when needed.
- Every interactive element is keyboard reachable and has an accessible name.
- An LMS has a legal accessibility bar in many places. Target WCAG 2.2 AA.

## Tooling

- ESLint (with `typescript-eslint`, `react-hooks`, `jsx-a11y`) and Prettier must pass before committing.
- The Vite dev server proxies `/api` to the local API, so no CORS is needed in dev.
