---
paths:
  - "api/**"
  - "web/src/api/**"
---

# API Design

The HTTP API is a product consumed by `web/` and possibly others later. Keep it consistent and predictable.

## Resources and URLs

REST is the inspiration, not a strict rulebook. Above all, a URL should read explicitly: anyone looking at it should be able to tell what every value means.

- **The path names the resource; query parameters supply the values.** Paths are static nouns. Identifiers and filters go in named query parameters, not path segments.

  | Do | Don't |
  |---|---|
  | `GET /api/courses?courseId=c-123` | `GET /api/courses/c-123` |
  | `GET /api/lessons?courseId=c-123` | `GET /api/courses/c-123/lessons` |
  | `GET /api/enrolments?courseId=c-123&learnerId=l-456` | `GET /api/courses/c-123/learners/l-456/enrolment` |
  | `DELETE /api/lessons?lessonId=ls-789` | `DELETE /api/lessons/ls-789` |

- **No path parameters.** Paths stay flat and contain no `{placeholders}`, so the route table reads as a list of resources.
- Use plural nouns for resources (`/courses`, `/lessons`, `/enrolments`). Avoid nesting: express relationships with query parameters (`/lessons?courseId=`), not deeper paths.
- Name parameters for what they are (`courseId`, `learnerId`, `status`), never generic names like `id` or `q`. Use the same name for the same concept across every endpoint.
- **Single-item vs collection.** A request that identifies one resource by its ID parameter (`GET /courses?courseId=c-123`) returns that item, or `404`. A request with no ID parameter, or with only filter parameters, returns a paginated collection (which may be empty).
- Use kebab-case for multi-word path segments; camelCase for JSON fields and query params.
- Use HTTP methods for their standard meaning. `GET` is safe and `PUT`/`DELETE` are idempotent. Model actions as resources (`POST /enrolments` with `courseId` and `learnerId` in the body), not as verbs in the URL.
- Commands (`POST`/`PUT`) carry their data in the JSON body. Query parameters on commands only identify the target (`PUT /courses?courseId=c-123`).
- Prefix with `/api` so the web app and API can share an origin.

## Requests and responses

- JSON in, JSON out. Set `Content-Type` explicitly via lenses.
- `201 Created` with a `Location` header for creates (`Location: /api/courses?courseId=c-123`); `204 No Content` for deletes.
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
