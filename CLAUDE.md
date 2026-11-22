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
