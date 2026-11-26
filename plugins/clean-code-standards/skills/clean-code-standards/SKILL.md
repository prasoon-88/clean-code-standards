---
name: clean-code-standards
description: Use when writing, refactoring or reviewing code. Enforces scoped UPPER_SNAKE_CASE const objects, export-only-if-imported, no workarounds, defensive boundaries, idiomatic classes and patterns, switch/lookup choice, and small diffs.
---

# Clean Code Standards

## 0. How to apply this skill

- **Scope: the lines you add or change.** Apply every rule to new and modified code. In existing code you only touch, *mention* violations you notice; don't fix them unless asked. This keeps diffs small (rule 1.9).
- **Project rules win.** If the project's `CLAUDE.md`, `AGENTS.md`, linter, formatter or style guide says something different, follow the project and point out the conflict once.
- **Match the surroundings.** File names, quote style, semicolons, indentation and comment density follow the sibling files and the project formatter. The examples below show *structure*, not formatting.
- **Deviation rule.** If a rule would make a specific piece of code worse, deviate and say why in one line. Never ignore a rule silently.
- **Reference files.** For general practices (naming, functions, async, logging, security, testing) read `references/general-practices.md`. For frontend, framework, API, database or git work read `references/domain-rules.md`. Read them only when the task touches those areas.
- **Before finishing**, run the checklist in section 2.

---

## 1. Core rules

### 1.1 Patterns: recognise, don't impose
Use a pattern only when the problem already has its shape. Never add one "for later".

| If you see… | Use |
|---|---|
| Branching on a value to pick *data* (value → value) | Const lookup object (1.3) |
| Branching on one value to pick *behaviour*, few cases, one place | `switch` / `match` (1.7) |
| The same behaviour choice repeated across many call sites, or new variants keep arriving | **Strategy** (map of handlers or interface + implementations) |
| Object creation that depends on input or config | **Factory** |
| Many optional constructor parameters | **Builder** or an options object |
| A third-party or legacy API used in several places | **Adapter** |
| Data access mixed into business logic | **Repository** |
| A function that creates its own collaborators (DB, HTTP client, clock) | **Dependency injection**: pass them in |
| Several consumers reacting to one event | **Observer / pub-sub** (or the framework's event system) |

Avoid Singleton unless the resource is truly process-wide (config, logger); prefer injection.

### 1.2 Reuse without speculation
- **Duplicated knowledge — extract on the 2nd occurrence.** A business rule, magic value, validation rule or formula must have one source of truth as soon as it appears twice.
- **Duplicated logic — extract on the 3rd occurrence** (rule of three). Two similar blocks may diverge; wait for the third before abstracting.
- **Generalise only for a real second caller.** Write the specific version first; add parameters or generics when another caller actually needs them.
- Prefer pure functions (inputs → output) over hidden global state.
- Put shared code in the project's existing shared location, grouped by domain. No dumping-ground `utils` file.

### 1.3 Constants: UPPER_SNAKE_CASE, const objects, right scope

**No meaningful inline literals.** Status strings, roles, routes, error codes, limits, timeouts, URLs, storage keys, event names and feature flags live in named constants. Allowed inline: `0`, `1`, `-1`, `''`, `true`/`false` in their obvious sense.

**Naming:** constants and const objects are `UPPER_SNAKE_CASE`, and so are the keys of const objects you define, **except** keys that must match an external contract (API payload fields, query params, theme/`sx` objects, third-party config). Keep those keys exactly as the contract requires.

**Group related values in an immutable const object** and derive types from it:

```ts
const ORDER_STATUS = {
  PENDING: 'pending',
  SHIPPED: 'shipped',
  CANCELLED: 'cancelled',
} as const

type OrderStatus = (typeof ORDER_STATUS)[keyof typeof ORDER_STATUS]

// ❌ if (order.status === 'shipped')
// ✅ if (order.status === ORDER_STATUS.SHIPPED)
```

Immutability per language: `as const` (TS), `Object.freeze` (JS), `Final` / `Enum` / frozen dataclass (Python), `enum` / `static final` (Java, Kotlin), `const` / `iota` (Go).

**Not constants:** colours, spacing, fonts and breakpoints come from the theme or design-system tokens. Never build a parallel `COLORS` object. User-facing text goes through the project's i18n system if it has one.

**Scope: place each constant at the level of its users, no wider and no narrower.**

| Used by | Lives in | Exported? |
|---|---|---|
| One file | Top of that file, right after imports | No |
| Several files in one feature/module | That feature's constants file | Yes, within the feature |
| Several features / whole app | The project's global constants location | Yes |

- Never declare a constant inside a function, loop or component body (it is re-created on every call/render). A value computed from arguments is a local variable, not a constant.
- **Promote** when a second file needs it: move it, never copy it.
- **Demote** when only one file still uses it: move it back and remove the `export`.
- Name constants files the way sibling files are named in that project (e.g. `constant.ts`, `orderConstants.ts`, `order.constants.ts`).
- Environment values are read once, validated, and exposed through a global config object. **No silent fallbacks**; a missing required value is a startup error:

```ts
// global config module — read env the project's way (process.env, import.meta.env, os.environ…)
const readRequired = (key: string, value: string | undefined): string => {
  if (!value) throw new Error(`Missing required env var: ${key}`)
  return value
}

export const API_CONFIG = {
  BASE_URL: readRequired('API_BASE_URL', import.meta.env.VITE_API_BASE_URL),
  TIMEOUT_MS: 5000,
} as const
```

### 1.4 Export only what another file imports
- Default to not exported. Add `export` only after confirming an importer exists (search the codebase).
- Remove the `export` when the last importer goes away.
- Applies to functions, classes, types and constants.
- Exceptions: a package's public API (entry point / index) and test-only exports when the project allows them.

### 1.5 Classes where the idiom fits
- Use a class when the language or framework favours it **and** the data has invariants to protect (e.g. a domain model, a service with injected dependencies, a client wrapper). Keep state private; validate in the constructor.
- Use plain functions and modules otherwise. In **React**, state and behaviour live in hooks, components and plain modules, never in classes; follow Redux Toolkit / React Query idioms where the project uses them.
- Prefer composition over inheritance; inherit only for a true "is-a".
- Never wrap a single stateless function in a class.

### 1.6 No workarounds
- Fix the root cause, not the symptom.
- Forbidden: `setTimeout` to dodge races, empty `catch`, `@ts-ignore` / `any` / `# type: ignore` to silence types, special cases for one input, copy-pasted patches, inline lint disables, retries that hide a bug, silent default values that mask missing config.
- If the real fix is out of scope, **stop and tell the user** what the real fix is. If they accept a temporary measure, mark it in the project's TODO format; if none exists, `TODO(<ticket-id>): <real fix>`. Never a TODO without a ticket or owner.

### 1.7 Switch, lookup or Strategy (one decision)
- **Value → value** (labels, config, mappings): const lookup object typed with `Record<Key, Value>`.
- **Value → behaviour, 3+ cases, one place:** `switch` / `match` on const-object members (never raw literals), with an exhaustive `default`.
- **Behaviour varies across many call sites or keeps growing:** Strategy (1.1).
- **1–2 branches, ranges or boolean conditions:** `if` / early return.

```ts
const STATUS_LABEL: Record<OrderStatus, string> = {
  [ORDER_STATUS.PENDING]: 'Awaiting shipment',
  [ORDER_STATUS.SHIPPED]: 'On the way',
  [ORDER_STATUS.CANCELLED]: 'Cancelled',
}

switch (status) {
  case ORDER_STATUS.PENDING:
    return handlePending(order)
  case ORDER_STATUS.SHIPPED:
    return handleShipped(order)
  case ORDER_STATUS.CANCELLED:
    return handleCancelled(order)
  default: {
    const exhaustiveCheck: never = status
    throw new Error(`Unhandled status: ${exhaustiveCheck}`)
  }
}
```

Keep each `case` short; delegate to a function past a few lines.

### 1.8 Defend at the boundaries, trust the types inside
Defensive code handles problems **explicitly**; it never hides them (that is a workaround, 1.6).

**Boundaries** (exported/public functions, API handlers, CLI entry points, message consumers, env/config, data from network, files, DB or third parties):
- Validate type, presence, range, length and format with guard clauses; fail fast with a specific error.
- Validate external data against a schema using **the project's existing validator** (Zod, Pydantic, etc.). Don't add a validation library without asking.
- Cap user-controlled sizes (payload, file size, item count, recursion depth).

**Everywhere:**
- Handle `null`/`undefined`/empty deliberately; use `?.` and `??` (not `||` when `0`, `false` or `''` are valid).
- Guard indexes, division by zero and loop termination.
- Don't mutate arguments; return copies or readonly views of internal state.
- Every network, DB or file call has a timeout, explicit error handling and resource cleanup (`finally`, `with`, `using`, `defer`). Retry only idempotent operations, capped, with backoff.
- Safe defaults: deny by default; never leak secrets or stack traces to end users.
- Guard shared mutable state; make handlers idempotent where events can repeat.

**Inside** private helpers whose callers are already validated and typed, don't re-check arguments.

```ts
// exported = boundary, so it validates
const PAGINATION = { MIN_PAGE: 1, MIN_PAGE_SIZE: 1, MAX_PAGE_SIZE: 100 } as const

export function getPage<T>(items: readonly T[], page: number, pageSize: number): T[] {
  if (!Number.isInteger(page) || page < PAGINATION.MIN_PAGE) {
    throw new RangeError(`page must be an integer >= ${PAGINATION.MIN_PAGE}, got ${page}`)
  }
  if (
    !Number.isInteger(pageSize) ||
    pageSize < PAGINATION.MIN_PAGE_SIZE ||
    pageSize > PAGINATION.MAX_PAGE_SIZE
  ) {
    throw new RangeError(
      `pageSize must be ${PAGINATION.MIN_PAGE_SIZE}-${PAGINATION.MAX_PAGE_SIZE}, got ${pageSize}`,
    )
  }
  return sliceRange(items, (page - 1) * pageSize, pageSize)
}

// private helper: callers already validated, so no checks
function sliceRange<T>(items: readonly T[], start: number, count: number): T[] {
  return items.slice(start, start + count)
}
```

### 1.9 Small, focused changes
- Touch only what the task needs: no drive-by refactors, renames or reformatting.
- Mention unrelated problems instead of fixing them.
- One logical change per commit/PR.

### 1.10 Comments and docs
- Match the comment density of the file you're in.
- Comments explain *why*, not *what*. No noise comments.
- Doc-comment exported APIs when the name and types don't make usage obvious (side effects, thrown errors, units, invariants), or when the project requires it.

---

## 2. Checklist (for added or changed lines)

**Constants & exports**
- [ ] No meaningful inline literals; related values in immutable `UPPER_SNAKE_CASE` const objects (external-contract keys exempt).
- [ ] Each constant at the scope of its users; none inside functions or components; nothing duplicated; colours/text from theme/i18n.
- [ ] No silent env/config fallbacks.
- [ ] Every new `export` has an importer; dead exports removed.

**Design**
- [ ] Patterns used only where the problem has their shape; nothing speculative.
- [ ] Knowledge extracted on 2nd use, logic on 3rd; no premature generics.
- [ ] Classes only where idiomatic and protecting invariants; hooks/modules in React.
- [ ] Lookup / `switch` / Strategy chosen per 1.7; exhaustive `default`.

**Safety**
- [ ] No workarounds; any accepted TODO has a ticket.
- [ ] Boundaries validated with the project's validator; no re-checks inside private helpers.
- [ ] External calls have timeouts, error handling and cleanup.

**Scope & verification**
- [ ] Diff limited to the task; pre-existing violations mentioned, not fixed.
- [ ] Comments match file density.
- [ ] Format, lint, **typecheck** and **tests** run and passing (where the project has them).
- [ ] Any deviation explained in one line.

## 3. Reviewing code

Report findings grouped by rule ID: `1.x` for core rules, `G.x` for `references/general-practices.md`, `D.x` for `references/domain-rules.md`. Give file and line for each, separate new issues from pre-existing ones, then offer fixes.
