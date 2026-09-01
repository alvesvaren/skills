# alve-skills

Claude Code plugin with opinionated conventions for the stack I actually use.

| Skill | Covers |
| --- | --- |
| `typescript-best-practices` | Types, Zod at trust boundaries, adapters, control flow, dependencies, async |
| `react-best-practices` | React 19 + Vite + TanStack Query + Tailwind v4 conventions |
| `testing-best-practices` | Vitest integration net, red-green TDD, Playwright E2E, edge mocking |
| `backend-best-practices` | Hono, layers, contracts (tRPC/OpenAPI), AppError + onError, Drizzle |

## Install

From GitHub:

```
/plugin marketplace add alvesvaren/skills
/plugin install alve-skills@alve-skills
```

For local development:

```sh
claude --plugin-dir /path/to/this/repo
```

then `/reload-plugins` after edits.

The skills are model-invoked: Claude applies them automatically when writing matching code. They can also be invoked manually, e.g. `/alve-skills:testing-best-practices`.
