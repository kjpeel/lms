---
paths:
  - "api/**"
  - "web/src/api/**"
---

# API Design

The HTTP API is a product consumed by `web/` and possibly others later. Keep it consistent and predictable.

## Resources and URLs

- Use plural nouns for collections and IDs for items: `/courses`, `/courses/{courseId}`, `/courses/{courseId}/lessons`.
- Use kebab-case for multi-word path segments; camelCase for JSON fields and query params.
- Use HTTP methods for their standard meaning. `GET` is safe and `PUT`/`DELETE` are idempotent. Model actions that don't fit as sub-resources (`POST /courses/{id}/enrolments`), not verbs in the URL.
- Prefix with `/api` so the web app and API can share an origin.

## Requests and responses

- JSON in, JSON out. Set `Content-Type` explicitly via lenses.
- `201 Created` with a `Location` header for creates; `204 No Content` for deletes.
- Use ISO-8601 UTC timestamps (`2026-11-22T10:15:30Z`).
- Represent IDs as opaque strings to clients.
- Paginate collections from day one: `?limit=&cursor=` and a `next` cursor in the response.

## Errors

- Use RFC 9457 Problem Details (`application/problem+json`) with `type`, `title`, `status`, `detail`, plus an `errors` array for validation failures.
- Status codes: `400` for malformed or invalid input, `401` for unauthenticated, `403` for not allowed, `404` for not found (also when the caller isn't allowed to know the resource exists), `409` for conflict or state violation, `422` for a well-formed request that breaks a business rule, `500` for a bug.
- Never leak stack traces, SQL or internal class names.

## Contract

- The OpenAPI spec is generated from http4k contract routes and served at `/api/openapi.json`.
- An approval test pins the spec, so any API change appears in the PR diff.
- `web/` generates its TypeScript API types from this spec. Don't hand-write request/response types in the frontend.
- Breaking changes (removing fields, renaming, changing semantics) need a deliberate plan. Add the new shape first, migrate the clients, then remove the old one.
