# Clean Code Standards — a Claude skill

A skill that makes Claude write, refactor and review code using consistent, strict best practices.

## What it enforces

**Core rules (always loaded)**
1. Design patterns as a recognition aid — use one only when the problem already has its shape
2. Reuse without speculation — extract duplicated knowledge on the 2nd use, duplicated logic on the 3rd
3. Constants in `UPPER_SNAKE_CASE` const objects, each placed at the scope of its users (file / feature / global), no silent env fallbacks
4. Export a function, type or constant only if another file imports it
5. Classes only where the language idiom fits and data has invariants (hooks and modules in React)
6. No workarounds — fix the root cause, or stop and tell the user
7. One decision rule for lookup object vs `switch` vs Strategy
8. Defend at the boundaries with the project's validator, trust the types inside
9. Small, focused diffs — rules apply to lines you add or change
10. Comments match the file's density

**Loaded only when relevant** (`references/`): general practices — SOLID, naming, functions, async, logging, dependencies, security, testing, performance — and domain rules for frontend/accessibility, React, Node, Python, Java/Kotlin, API design, databases and git.

Project rules (`CLAUDE.md`, `AGENTS.md`, linters) always win, and a checklist — including format, lint, typecheck and tests — runs before Claude finishes.

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
