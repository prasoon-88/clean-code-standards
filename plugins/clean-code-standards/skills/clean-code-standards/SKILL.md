---
name: clean-code-standards
description: Use when writing, refactoring or reviewing code to enforce design patterns, reusability, defensive coding, scoped const objects, naming, minimal exports, classes, switch-case and no workarounds.
---

# Clean Code Standards

Apply these rules to every piece of code you write, change or review. When a rule conflicts with an existing project convention (linter config, style guide, CLAUDE.md), the project convention wins; point out the conflict once.

**Deviation rule:** if following a rule would make the code worse in a specific case (a class is overkill, a switch is less clear than a map), you may deviate — but say so in one line with the reason. Never silently ignore a rule.

Before finishing, run through the **Checklist** at the bottom and fix any violations.

---

## 1. Core rules (mandatory)

### 1.1 Use proven design patterns
- Pick the pattern that fits the problem, not the fanciest one. Common fits:
  - **Strategy** – interchangeable algorithms/behaviours (replaces long if/else on a "type").
  - **Factory / Factory Method** – object creation that depends on input or config.
  - **Builder** – objects with many optional parameters.
  - **Adapter** – wrapping third-party or legacy APIs behind your own interface.
  - **Facade** – a simple entry point over a complex subsystem.
  - **Observer / Pub-Sub** – event-driven decoupling.
  - **Repository** – isolating data access from business logic.
  - **Dependency Injection** – pass collaborators in; don't construct them inside.
  - **Singleton** – only for truly global, stateless or shared resources (config, logger); prefer DI otherwise.
- Name the pattern in a short comment or class name when it isn't obvious (e.g. `PaymentStrategy`, `UserRepository`).

### 1.2 Build reusable code
- **DRY**: if logic appears twice, extract it into a function, class or module.
- Write functions that take inputs and return outputs; avoid hidden dependencies on global state.
- Make utilities generic (parameters, generics/templates) instead of hard-coding one use case.
- Put shared helpers in a clearly named shared module (`utils/`, `common/`, `lib/`), grouped by domain, not a single dumping-ground file.

### 1.3 Constants: UPPER_SNAKE_CASE, const objects, correct scope

**Naming:** every true constant is `UPPER_SNAKE_CASE` — module-level, global, class static constants, and constant objects (keys inside them too).

**No inline hard-coded values.** Every literal that carries meaning (status strings, roles, routes, error codes, limits, timeouts, URLs, keys, labels, event names, colours, feature flags) lives in a named constant.

**Group related hard-coded values in an immutable const object** so code is driven by data, not scattered literals:

```ts
const ORDER_STATUS = {
  PENDING: "pending",
  SHIPPED: "shipped",
  CANCELLED: "cancelled",
} as const;

type OrderStatus = (typeof ORDER_STATUS)[keyof typeof ORDER_STATUS];
```

```ts
// ❌ Bad
if (order.status === "shipped") res.status(200);

// ✅ Good
if (order.status === ORDER_STATUS.SHIPPED) res.status(HTTP_STATUS.OK);
```

- Make const objects immutable: `as const` (TS), `Object.freeze` (JS), `Final`/`Enum`/frozen dataclass (Python), `enum`/`static final` (Java/Kotlin), `const`/`iota` (Go).
- Derive types from the const object instead of repeating the values in a separate union type.
- Prefer **value-driven lookups** over branching where values map to values:
  ```ts
  const STATUS_LABEL: Record<OrderStatus, string> = {
    [ORDER_STATUS.PENDING]: "Awaiting shipment",
    [ORDER_STATUS.SHIPPED]: "On the way",
    [ORDER_STATUS.CANCELLED]: "Cancelled",
  } as const;
  ```
- Allowed inline literals: `0`, `1`, `-1`, `""`, `true`/`false` used in their obvious sense (loop start, increment, empty check).

**Scope: place each constant at the level where it is used — no wider, no narrower.**

| Used by | Where it lives | Exported? |
|---|---|---|
| One module (file) only | **Module level**: top of that file, right after imports | No |
| Several files in one feature/module folder | Feature constants file, e.g. `orders/order.constants.ts` | Yes, to that feature only |
| Multiple features / the whole app | **Global** constants folder, e.g. `src/constants/http.constants.ts` | Yes |

```ts
// src/constants/http.constants.ts  — GLOBAL: used app-wide
export const HTTP_STATUS = {
  OK: 200,
  CREATED: 201,
  BAD_REQUEST: 400,
  NOT_FOUND: 404,
  INTERNAL_ERROR: 500,
} as const;

// src/constants/api.constants.ts  — GLOBAL
export const API_CONFIG = {
  BASE_URL: process.env.API_BASE_URL ?? "http://localhost:3000",
  TIMEOUT_MS: 5000,
} as const;
```

```ts
// src/orders/order.service.ts  — MODULE LEVEL: used only in this file
import { HTTP_STATUS } from "../constants/http.constants";

const ORDER_RETRY = {
  MAX_COUNT: 3,
  BACKOFF_MS: 200,
} as const;

class OrderService { /* uses ORDER_RETRY and HTTP_STATUS */ }
```

Placement rules:
- **Module-level constants stay in their module.** Declare them at the top of the file after imports — never in a global constants file, never buried mid-file, and never inside a function or loop (that re-creates them on every call). The only exception is a value computed from function arguments, which is a local variable, not a constant.
- **Global constants stay global.** Define them once in the global constants folder and import them. Never redefine or copy a global value locally.
- **Promote** a module-level constant to the feature or global level as soon as a second file needs it — move it, don't duplicate it.
- **Demote** a feature/global constant back to module level when only one file still uses it, and remove its `export` (see 1.4).
- Values that differ per environment come from config/env and are exposed through a global const config object (like `API_CONFIG`).
- Group global constants by domain (`http.constants.ts`, `auth.constants.ts`), not one giant file.

### 1.4 Export a function only if another file imports it
- Default to **not exported** (private to the module).
- Export only when at least one other file actually imports it. Before adding `export`, confirm the importer exists (search the codebase).
- When removing the last importer, remove the `export` too.
- The same rule applies to constants: module-level constants are never exported (see 1.3).
- Exception: the module's intended public API (package entry point, library index) and test-only exports when the project explicitly allows them.

### 1.5 Abstract into classes where possible
- When data and the functions operating on it belong together, model them as a class.
- Keep state private; expose behaviour through methods.
- Use interfaces / abstract base classes to define contracts, and concrete classes to implement them.
- Prefer **composition over inheritance**; inherit only for a true "is-a" relationship.
- Don't force a class around a single stateless pure function — a plain function is fine there.

### 1.6 No workarounds
- Fix the **root cause**, not the symptom.
- Forbidden: `setTimeout` to dodge race conditions, swallowing errors with empty `catch`, `// @ts-ignore` / `any` / `# type: ignore` to silence types, hard-coded special cases for one input, copy-pasted patches, disabling lint rules inline, retry loops hiding a real bug.
- If a proper fix is genuinely out of scope, stop and tell the user what the real fix is instead of quietly shipping a hack. If they explicitly accept a temporary measure, mark it `// TODO(reason): ...` with the real fix described.

### 1.7 Prefer switch-case where it fits
- Use `switch` (or `match` in Python/Rust/Kotlin) when branching on a **single value** against 3+ discrete cases (enums, const-object values, status codes).
- Switch on const-object members, never raw literals (`case ORDER_STATUS.SHIPPED:`).
- Always include a `default` branch; for enums/const objects, make it exhaustive (e.g. TypeScript `never` check).
- Keep each case short — delegate to a function if a case grows beyond a few lines.
- When cases map value → value with no logic, use a const lookup object (1.3) or Strategy instead.
- Keep `if/else` for range checks, boolean conditions or 1–2 branches.

```ts
switch (status) {
  case ORDER_STATUS.PENDING:
    return handlePending(order);
  case ORDER_STATUS.SHIPPED:
    return handleShipped(order);
  case ORDER_STATUS.CANCELLED:
    return handleCancelled(order);
  default: {
    const exhaustiveCheck: never = status;
    throw new Error(`Unhandled status: ${exhaustiveCheck}`);
  }
}
```

### 1.8 Code defensively
Assume every input, dependency and environment can misbehave. Defensive code is **explicit handling**, never silent hiding (that's a workaround — see 1.6).

**Validate inputs**
- Check arguments at every public/exported function, API handler, CLI entry and message consumer: type, presence, range, length, format.
- Use guard clauses at the top of the function and fail fast with a clear, specific error.
- Validate external data (HTTP bodies, files, env vars, DB rows, third-party responses) against a schema (e.g. Zod, Joi, Pydantic, JSON Schema) before using it.

**Handle null / undefined / empty**
- Never assume a value exists. Use optional chaining (`?.`), nullish coalescing (`??`), or explicit checks.
- Treat empty strings, empty arrays, `0`, `NaN` and missing keys as distinct cases and handle each deliberately.
- Use `??` rather than `||` for defaults when `0`, `false` or `""` are valid values.

**Check boundaries**
- Guard array/list indexes, division by zero, integer overflow, string lengths, pagination limits and loop termination.
- Cap sizes of user-controlled input (payload size, file size, item counts, recursion depth).

**Protect state**
- Don't mutate arguments; copy them. Return copies (or readonly views) of internal collections.
- Make objects immutable where practical (`readonly`, `Object.freeze`, frozen dataclasses).
- Keep class invariants valid after every method; validate in constructors so invalid objects can't exist.

**External calls and I/O**
- Every network/DB/file call has a timeout and explicit error handling.
- Retry only idempotent operations, with a capped count and backoff — never as a cover for a bug.
- Always release resources: `finally`, `try-with-resources`, `with`, `using`, `defer`.
- Handle partial failures and unexpected response shapes.

**Fail safely**
- Prefer fail-fast with a clear error over continuing in a corrupted state.
- Use safe defaults (deny by default for permissions, closed by default for access).
- Use assertions for "impossible" internal states and exhaustive checks for enums (see 1.7).
- Error messages help developers but never leak secrets, stack traces or internal details to end users.

**Concurrency**
- Guard shared mutable state (locks, atomic operations, transactions) and avoid check-then-act races.
- Make handlers idempotent where events or requests may repeat.

```ts
// module-level: only this file paginates
const PAGINATION = { MIN_PAGE: 1, MIN_PAGE_SIZE: 1, MAX_PAGE_SIZE: 100 } as const;

function getPage<T>(items: readonly T[], page: number, pageSize: number): T[] {
  if (!Array.isArray(items)) {
    throw new TypeError("items must be an array");
  }
  if (!Number.isInteger(page) || page < PAGINATION.MIN_PAGE) {
    throw new RangeError(`page must be a positive integer, got ${page}`);
  }
  if (
    !Number.isInteger(pageSize) ||
    pageSize < PAGINATION.MIN_PAGE_SIZE ||
    pageSize > PAGINATION.MAX_PAGE_SIZE
  ) {
    throw new RangeError(`pageSize must be 1-${PAGINATION.MAX_PAGE_SIZE}, got ${pageSize}`);
  }
  const start = (page - 1) * pageSize;
  return items.slice(start, start + pageSize);
}
```

Don't over-defend: inside a private helper whose callers are already validated and typed, re-checking every argument adds noise. Validate at the boundaries, trust the types inside.

### 1.9 Keep changes small and focused
- Touch only what the task requires. No drive-by refactors, renames or reformatting of unrelated code.
- If you spot an unrelated problem, mention it to the user instead of fixing it in the same change.
- One logical change per commit/PR so diffs stay reviewable.

---

## 2. Additional best practices (also enforced)

### Principles
- **SOLID**: Single responsibility, Open/closed, Liskov substitution, Interface segregation, Dependency inversion.
- **KISS** – simplest solution that works.
- **YAGNI** – don't build features or abstractions nobody needs yet.
- **Separation of concerns** – UI, business logic and data access live in different layers.
- **Law of Demeter** – talk to direct collaborators, avoid `a.b().c().d()` chains.

### Naming
- Variables & functions: `camelCase` (or the language's idiom, e.g. `snake_case` in Python).
- Classes, interfaces, types, enums: `PascalCase`.
- Constants and const objects: `UPPER_SNAKE_CASE` (keys inside const objects too).
- File names: follow project convention (`kebab-case` by default); constants files end in `.constants.<ext>`.
- Booleans read as questions: `isActive`, `hasAccess`, `shouldRetry`.
- Functions start with a verb: `getUser`, `calculateTotal`, `sendEmail`.
- No vague names (`data`, `temp`, `obj`, `handle2`) and no unexplained abbreviations.

### Functions
- One function, one job; aim for under ~30 lines.
- At most 3 parameters; group more into an options object / dataclass.
- Use **early returns / guard clauses** instead of deep nesting (max ~3 levels).
- Avoid boolean flag parameters that switch behaviour — split into two functions.
- Prefer pure functions; isolate side effects.

### Immutability & types
- Use `const` / `final` / `readonly` by default; mutate only when needed.
- Use strict typing; no `any` / untyped dicts in public signatures.
- Use const objects or enums (1.3) instead of raw strings for fixed sets of values.

### Error handling
- Never swallow errors; handle them or rethrow with context.
- Use custom error classes for domain errors; keep error codes in a const object at the right scope (1.3).
- Fail fast with clear messages (details in 1.8).

### Async
- Every promise is awaited, returned, or explicitly handled — no floating promises.
- Never use `async` callbacks inside `forEach`; use `for...of` (sequential) or `Promise.all` / `Promise.allSettled` (parallel).
- Run independent async work in parallel; limit concurrency for large batches.
- Support cancellation for long-running work (`AbortController`, context, cancellation tokens).
- Don't mix callbacks and promises in the same API.

### Logging
- Use the project logger with levels (`debug`, `info`, `warn`, `error`) and structured fields, not string concatenation.
- Never log secrets, tokens, passwords or personal data; mask when needed.
- No leftover `console.log` / `print` / debugger statements.
- Log errors once, at the layer that handles them, with enough context to debug (IDs, operation).

### Formatting & linting
- Code must pass the project's formatter and linter (Prettier, ESLint, Black, Ruff, gofmt, etc.) with zero warnings.
- Run them before declaring the work done when they're available; fix issues rather than disabling rules.

### Dependencies
- Don't add a library for something a few lines of code can do.
- Prefer well-maintained, widely used packages; check licence and last release.
- Pin or lock versions; update the lockfile with the change.
- Wrap third-party libraries behind an Adapter when they're used in many places.

### Structure
- File order: imports → module-level constants → types → private helpers → main class/functions → exports.
- One main class / concern per file.
- Order imports: standard library → third-party → internal; remove unused imports.
- No circular dependencies.
- Delete dead code, commented-out code and unused variables — don't leave them behind.

### Comments & docs
- Code should explain *what*; comments explain *why*.
- Doc-comment every exported function/class (params, return, throws).
- No noise comments (`// increment i`).

### Security & config
- No secrets, keys or credentials in code — use environment variables / config.
- Sanitize and escape all external input; use parameterized queries, never string-built SQL/shell commands.
- Keep environment-specific values in config, not in logic; validate config at startup.

### Testability
- Design for testing: inject dependencies so they can be mocked.
- Add or update tests alongside changed behaviour when the project has tests, including edge cases (null, empty, boundaries, failures).

### Performance (without premature optimization)
- Avoid obvious inefficiencies (N+1 queries, repeated work in loops, unnecessary re-renders).
- Optimize only with evidence; readability first.

---

## 3. Domain-specific rules (apply the sections that match the code)

### Frontend & accessibility
- Use semantic HTML (`button`, `nav`, `main`, `label`) instead of clickable `div`s.
- Every input has a label; every meaningful image has `alt` text; decorative images use `alt=""`.
- Everything works by keyboard with a visible focus state; meet WCAG AA colour contrast.
- Use ARIA only when native elements can't express the meaning.

### Framework conventions
- Follow the framework's official rules, for example:
  - **React**: Rules of Hooks, stable `key`s (never array index for dynamic lists), complete effect dependencies, module-level constants outside the component (not re-created each render).
  - **Node.js**: handle `unhandledRejection`, never block the event loop with sync I/O in request paths.
  - **Python**: type hints on all public functions, `pathlib` over string paths, context managers for resources.
  - **Java/Kotlin**: `Optional`/nullable types instead of returning null, `try-with-resources`/`use`.
- Use the framework's built-in way before writing your own.

### API design
- Consistent response shape for success and error (e.g. `{ data }` / `{ error: { code, message } }`).
- Correct HTTP methods and status codes, taken from the global `HTTP_STATUS` const object.
- Version the API (`/v1/...`) and don't make breaking changes within a version.
- Paginate list endpoints with a capped page size; validate every request body and query.
- Use consistent naming for routes (plural nouns, `kebab-case`) and fields.

### Database
- Every schema change is a migration in version control — never manual edits.
- Use transactions for writes that span multiple tables or must succeed together.
- Index columns used in `WHERE`, `JOIN` and `ORDER BY` on large tables.
- Parameterized queries or the ORM only; avoid N+1 queries (eager load / batch).
- Select only needed columns; never `SELECT *` in production code.

### Git hygiene
- Commit messages follow Conventional Commits: `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:` + short imperative summary.
- One logical change per commit; don't mix formatting with behaviour changes.
- Never commit secrets, `.env` files, build output or local config.

---

## 4. Checklist (run before finishing)

- [ ] Appropriate design pattern used; no long if/else chains on type.
- [ ] No duplicated logic; shared code extracted and reusable.
- [ ] All constants `UPPER_SNAKE_CASE`; hard-coded values grouped in immutable const objects; no magic literals inline.
- [ ] Module-only constants at the top of their module (not exported, not inside functions); shared ones at feature level; app-wide ones in the global constants folder; nothing duplicated.
- [ ] Every `export` has at least one importer in another file (or is public API).
- [ ] Related data + behaviour grouped into classes; composition over inheritance.
- [ ] No workarounds, hacks, ignored types, empty catches or disabled lint rules.
- [ ] `switch`/`match` on const-object values for 3+ cases, with an exhaustive `default`.
- [ ] Inputs validated at boundaries; null/empty/boundary cases handled.
- [ ] External calls have timeouts, error handling and resource cleanup.
- [ ] No argument mutation; invariants protected; safe defaults used.
- [ ] Change is small and focused; no unrelated edits.
- [ ] No floating promises; async used correctly.
- [ ] Structured logging; no secrets logged; no debug prints left.
- [ ] Formatter and linter pass with zero warnings.
- [ ] No unnecessary new dependencies; versions locked.
- [ ] Domain rules applied (accessibility, framework, API, DB, git) where relevant.
- [ ] SOLID / KISS / YAGNI respected; naming conventions followed; functions small with early returns.
- [ ] No dead code, unused imports, or secrets.
- [ ] Exported items documented; tests (incl. edge cases) updated if applicable.
- [ ] Any rule deviation explained in one line.

When reviewing existing code, report violations grouped by rule number with file and line, then offer fixes.
