# node-ci

Fledge flow for Node.js / TypeScript projects: install, lint, type-check, test, and build.

## Flows

### `ci` — Full sequential CI
Sequential install → lint → typecheck → test → build. Reliable ordering for CI pipelines.

```bash
fledge run ci
```

### `check` — Parallel lint + typecheck
Installs deps, then runs lint and typecheck in parallel before testing. Faster for local dev.

```bash
fledge run check
```

### `build-only` — Type-safe build
Install → typecheck → build. Skips lint and tests; useful for release artifact generation.

```bash
fledge run build-only
```

## Tasks

| Task | Command | Deps | Notes |
|------|---------|------|-------|
| `install` | `npm ci` | — | Clean reproducible install |
| `lint` | `npm run lint` | `install` | ESLint / Biome |
| `typecheck` | `npm run typecheck` | `install` | `tsc --noEmit` |
| `test` | `npm test` | `install` | Jest / Vitest |
| `build` | `npm run build` | `typecheck` | Production bundle; `NODE_ENV=production` |

## Patterns demonstrated

- **Inline task deps** — `lint`, `typecheck`, `test` all depend on `install`, so `fledge run lint` always installs first
- **Parallel step** — `{ parallel = ["lint", "typecheck"] }` in the `check` flow cuts wall-clock time
- **Task env** — `NODE_ENV = "production"` scoped to `build` only
- **Build-only flow** — subset of tasks for artifact generation without full test overhead
