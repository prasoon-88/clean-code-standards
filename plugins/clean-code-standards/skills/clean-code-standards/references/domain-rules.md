# Domain rules (D)

Read the section that matches the code you're working on. Project rules win; apply to added or changed lines only.

## D.1 Frontend and accessibility
- Semantic HTML (`button`, `a`, `nav`, `main`, `label`) instead of clickable `div`s.
- Every input has an associated label; meaningful images have `alt` text; decorative images use `alt=""`.
- Everything works by keyboard with a visible focus state; meet WCAG AA contrast (use theme tokens, never hard-coded colours).
- ARIA only when no native element expresses the meaning.
- User-facing text goes through the project's i18n system when it has one.

## D.2 React
- Follow the Rules of Hooks; complete effect dependency arrays; no effects for derived state.
- Stable `key`s from data IDs; never the array index for lists that change.
- Constants and pure helpers live at module level, outside the component.
- State and logic in hooks and plain modules, not classes.
- Use the project's state and data tools (Redux Toolkit slices, React Query hooks, etc.) the way the codebase already does.
- Style through the theme / design system (`sx`, styled components, tokens), not inline colour or spacing literals.

## D.3 Node.js
- Never block the event loop with sync I/O or CPU-heavy work in request paths.
- Handle `unhandledRejection` and shut down gracefully on `SIGTERM`.
- Stream large payloads instead of buffering them.

## D.4 Python
- Type hints on public functions; run the project's type checker.
- `pathlib` over string paths; context managers for resources.
- No mutable default arguments.

## D.5 Java / Kotlin
- `Optional` / nullable types instead of returning `null`.
- `try-with-resources` / `use` for closeable resources.
- Immutable data classes / records for value objects.

## D.6 API design
- One response shape for success and error (e.g. `{ data }` / `{ error: { code, message } }`).
- Correct HTTP methods; status codes from a global `HTTP_STATUS` const object.
- Version the API; no breaking changes within a version.
- Paginate list endpoints with a capped page size; validate every body and query at the handler.
- Consistent route naming (plural nouns) and field casing matching the existing API.

## D.7 Database
- Every schema change is a migration in version control.
- Transactions for writes that must succeed together.
- Index columns used in `WHERE`, `JOIN` and `ORDER BY` on large tables.
- Parameterised queries or the ORM only; avoid N+1 (eager load or batch).
- Select only the columns you need.

## D.8 Git
- Follow the project's commit convention; default to Conventional Commits (`feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`).
- One logical change per commit; never mix formatting with behaviour changes.
- Never commit secrets, `.env` files, build output or local config.
