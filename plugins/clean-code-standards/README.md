# Clean Code Standards — a Claude skill

A skill that makes Claude write, refactor and review code using consistent, strict best practices.

## What it enforces

**Core rules**
1. Proven design patterns (Strategy, Factory, Builder, Adapter, Facade, Observer, Repository, DI)
2. Reusable, DRY code
3. Constants in `UPPER_SNAKE_CASE`, hard-coded values grouped in immutable const objects, each placed at the right scope (module / feature / global)
4. Export a function or constant only if another file imports it
5. Abstract related data + behaviour into classes (composition over inheritance)
6. No workarounds — fix the root cause
7. `switch` / `match` for 3+ discrete cases, with an exhaustive `default`
8. Defensive coding — validate inputs, handle null/empty/boundaries, timeouts, safe defaults
9. Small, focused changes

**Also covers:** SOLID / KISS / YAGNI, naming, function size, immutability & types, error handling, async, logging, formatting & linting, dependencies, file structure, comments, security & config, testability, performance, frontend accessibility, framework conventions (React, Node, Python, Java/Kotlin), API design, database and git hygiene — plus a 21-item checklist Claude runs before finishing.

See the full rules in [`plugins/clean-code-standards/skills/clean-code-standards/SKILL.md`](skills/clean-code-standards/SKILL.md).

## Install

### Claude Code (plugin marketplace — recommended)

```bash
claude plugin marketplace add prasoon-88/clean-code-standards
claude plugin install clean-code-standards@clean-code-standards
```

Or inside a Claude Code session:

```
/plugin marketplace add prasoon-88/clean-code-standards
/plugin install clean-code-standards@clean-code-standards
```

### Claude Code (manual)

Copy the skill folder into your skills directory:

```bash
git clone https://github.com/prasoon-88/clean-code-standards.git
cp -r clean-code-standards/plugins/clean-code-standards/skills/clean-code-standards ~/.claude/skills/        # all projects
# or
cp -r clean-code-standards/plugins/clean-code-standards/skills/clean-code-standards .claude/skills/          # current project only
```

### Claude app (claude.ai / Desktop)

1. Download [`dist/clean-code-standards.zip`](https://github.com/prasoon-88/clean-code-standards/raw/main/dist/clean-code-standards.zip).
2. Go to **Customize → Skills**, click **+**, then **Upload a skill**.
3. Upload the ZIP and make sure the skill is toggled on.

## Usage

Claude applies the skill automatically whenever it writes, refactors or reviews code. You can also call it directly:

```
/clean-code-standards review src/orders
```

If your project has its own conventions (linter config, style guide, `CLAUDE.md`), those take priority; Claude will point out any conflict once.

## Contributing

Issues and pull requests are welcome. Keep rules concrete and testable, and add an example when a rule isn't obvious.

## License

[MIT](LICENSE)
