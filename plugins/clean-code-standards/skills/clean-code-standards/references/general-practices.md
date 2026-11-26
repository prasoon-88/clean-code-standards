# General practices (G)

Read when the task involves these areas. Project rules win; apply to added or changed lines only.

## G.1 Principles
- **SOLID:** one responsibility per unit; extend without rewriting; subtypes honour their contracts; small interfaces; depend on abstractions you inject.
- **KISS:** the simplest solution that works.
- **YAGNI:** no features, parameters or abstractions without a current caller.
- **Separation of concerns:** UI, business logic and data access in separate layers.
- **Law of Demeter:** talk to direct collaborators; avoid `a.b().c().d()` chains.

## G.2 Naming
- Follow the language idiom: `camelCase` (JS/TS/Java), `snake_case` (Python/Rust) for variables and functions; `PascalCase` for classes, types and components; `UPPER_SNAKE_CASE` for constants (core rule 1.3).
- Booleans read as questions: `isActive`, `hasAccess`, `shouldRetry`.
- Functions start with a verb: `getUser`, `calculateTotal`.
- No vague names (`data`, `temp`, `obj`, `handle2`) or unexplained abbreviations.
- File names match sibling files.

## G.3 Functions
- One job each; around 30 lines or fewer is a smell threshold, not a hard limit.
- At most 3 positional parameters; group the rest into an options object.
- Early returns / guard clauses instead of nesting deeper than ~3 levels.
- No boolean flag parameters that switch behaviour; split into two functions.
- Isolate side effects from pure logic.

## G.4 Types and immutability
- `const` / `readonly` / `final` by default.
- Strict types; no `any` or untyped dicts in public signatures (use `unknown` + narrowing at boundaries).
- Const objects or enums for fixed sets of values (core rule 1.3).

## G.5 Error handling
- Never swallow errors; handle them or rethrow with context (`cause`).
- Custom error classes or error codes for domain errors; codes live in a const object at the right scope.
- Log an error once, at the layer that handles it.

## G.6 Async
- Every promise is awaited, returned or explicitly handled; no floating promises.
- No `async` callbacks in `forEach`; use `for…of` (sequential) or `Promise.all` / `Promise.allSettled` (parallel).
- Parallelise independent work; limit concurrency for large batches.
- Support cancellation for long-running work (`AbortController`, context, cancellation tokens).
- Don't mix callbacks and promises in one API.

## G.7 Logging
- Use the project's logger with levels and structured fields.
- Never log secrets, tokens, passwords or personal data.
- No leftover `console.log`, `print` or `debugger`.

## G.8 Dependencies
- Don't add a library for something a few lines can do; ask before adding any new dependency.
- Prefer well-maintained packages; check licence and last release.
- Commit the lockfile change with the dependency change.
- Wrap a third-party library behind an Adapter when it's used in many places.

## G.9 File structure
- Order: imports → module-level constants → types → private helpers → main code. Use inline `export` unless the project exports at the end.
- Imports: standard library → third-party → internal; no unused imports.
- One main concern per file; no circular dependencies.
- No dead code, commented-out code or unused variables in what you add.

## G.10 Security and config
- No secrets in code; read them from env/secret stores via the global config object.
- Parameterised queries; never build SQL or shell commands from strings.
- Escape output for its context (HTML, URLs, shell).
- Validate config at startup; fail on missing required values.

## G.11 Testing
- Inject dependencies so they can be replaced in tests.
- Add or update tests for changed behaviour, including edge cases: null/empty, boundaries, error paths.
- Tests assert behaviour, not implementation details.

## G.12 Performance
- Avoid obvious waste: N+1 queries, repeated work in loops, unnecessary re-renders.
- Optimise only with evidence (profile, benchmark); readability first.
