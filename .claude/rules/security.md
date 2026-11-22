# Security and Data Protection

An LMS holds personal data about learners (names, emails, progress, grades). Treat it accordingly.

## Secrets

- Never commit secrets (see CLAUDE.md). Config secrets come from environment variables via `http4k-config`.
- Never log secrets, tokens, passwords or full request bodies that may contain them.

## Authentication and authorisation

- Authentication happens in an http4k `Filter` at the edge. Downstream code receives an authenticated principal, not raw tokens.
- **Authorisation decisions live in the domain/application layer**, expressed as rules ("an instructor may edit only their own courses"). Don't scatter them as checks in route handlers.
- Deny by default. Every new route needs an explicit decision on who may call it, and a test that proves unauthorised callers get `401`/`403`.

## Input and output

- All input enters through lenses and value types (see `http4k.md`), so it's validated before it reaches the domain.
- Use parameterised queries only. Never build SQL by string concatenation.
- Don't render user-supplied HTML in the web app. If rich text is ever needed, sanitise it on the server.

## Personal data

- Collect only what a feature needs.
- Don't put personal data in logs, URLs or error messages. Log IDs, not emails.
- Use synthetic data in test fixtures, never real learner data.
