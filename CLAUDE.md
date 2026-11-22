# LMS

## Stack

- `/api`: Kotlin with [http4k](https://www.http4k.org/), built with Gradle
- `/web`: React and TypeScript, built with Vite

## Working rules

- Work on one small task at a time. Finish it before starting the next.
- Commit after each task, with a message that describes the change.
- Never commit secrets: no `.env` files, API keys, tokens, passwords or credentials. Keep config secrets in `.env` (gitignored) and commit a `.env.example` with placeholder values instead.
- Check `git status` before committing to make sure only intended files are staged.
- Don't commit directly to `main`. Do work on a feature branch, push it, and open a pull request with `gh pr create`.

## Engineering rules

Detailed rules live in `.claude/rules/`. Claude Code loads them automatically, and files with `paths:` frontmatter apply only to matching files.

- `architecture.md`: hexagonal architecture, layers, package layout (api)
- `http4k.md`: http4k patterns: app-as-a-function, lenses, filters, config, fakes (api)
- `kotlin-style.md`: Kotlin modelling and style conventions (api)
- `api-design.md`: URL, response, error and OpenAPI contract conventions
- `frontend.md`: React/TypeScript structure and conventions (web)
- `testing.md`: test pyramid, fakes over mocks, contract tests, tooling
- `dependencies.md`: approved stack, what to avoid, how to add or upgrade
- `security.md`: secrets, auth, input handling, personal data
