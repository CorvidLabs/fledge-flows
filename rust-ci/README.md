# rust-ci

Fledge lane for Rust projects: format check, Clippy linting, tests, and release build.

## Lanes

### `ci` — Full sequential CI
Runs fmt → lint → test → build in order. Fails fast on first error. Use this for PR checks.

```bash
fledge run ci
```

### `check` — Fast parallel pre-commit check
Runs fmt and lint in parallel, then test. Skips the full build — ideal for local iteration.

```bash
fledge run check
```

### `full` — Parallel lint + full build
Parallel fmt+lint, then test, then release build. Best for CI environments with multiple cores.

```bash
fledge run full
```

## Tasks

| Task | Command | Notes |
|------|---------|-------|
| `fmt` | `cargo fmt --check` | Fails if formatting is off |
| `lint` | `cargo clippy -- -D warnings` | Treats warnings as errors |
| `test` | `cargo test` | All unit and integration tests |
| `build` | `cargo build --release` | Release binary; depends on `lint` |

## Patterns demonstrated

- **Sequential lane** with `fail_fast`
- **Parallel step** `{ parallel = ["fmt", "lint"] }` for faster wall-clock time
- **Task deps** — `build` declares `deps = ["lint"]` so it always re-lints when run standalone
- **Task env** — `RUST_LOG = "info"` scoped to the build task only
