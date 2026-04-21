# fledge-lanes

A collection of example [fledge](https://github.com/CorvidLabs/fledge) lane configurations for common project types.

Each directory contains a `fledge.toml` with fully-documented tasks, lanes, parallel groups, dependencies, and fail-fast settings.

## Available Lanes

| Path | Description |
|------|-------------|
| [`rust-ci/`](rust-ci/) | Rust CI — fmt, clippy, test, release build with parallel fmt+lint |
| [`node-ci/`](node-ci/) | Node.js / TypeScript CI — lint, typecheck, test, production build |
| [`release/`](release/) | Release lane — version bump, changelog, git tag, npm publish, push |
| [`docker/`](docker/) | Docker build + push — lint Dockerfile, build, test image, push to registry |
| [`python/`](python/) | Python — lint (ruff), format (ruff), typecheck (mypy), test (pytest) |
| [`node-typescript/`](node-typescript/) | Node/TypeScript — build (tsc), lint (eslint), format (prettier), test (vitest) |
| [`rust/`](rust/) | Rust — clippy, test, fmt, build, release |
| [`go/`](go/) | Go — vet, test, build, staticcheck |

## Importing Lanes

### Search for lanes

```bash
fledge lane --search
```

This searches GitHub for repos tagged with the `fledge-lane` topic and lists available lane collections.

### Import a specific lane

```bash
# Import by repo and path
fledge lane --import CorvidLabs/fledge-lanes/rust

# Import into current project's fledge.toml
fledge lane --import CorvidLabs/fledge-lanes/python
```

Imported lanes are merged into your project's `fledge.toml`. Existing tasks/lanes with the same name are not overwritten unless you pass `--force`.

### Manual import

Copy the relevant `fledge.toml` into your project and adjust commands to match your toolchain:

```bash
curl -sL https://raw.githubusercontent.com/CorvidLabs/fledge-lanes/main/rust/fledge.toml \
  >> fledge.toml
```

## Lane Concepts

### Tasks

Tasks are the building blocks — named shell commands with optional dependencies, environment variables, and working directories:

```toml
[tasks.test]
cmd = "cargo test"
description = "Run test suite"

[tasks.build]
cmd = "cargo build --release"
deps = ["test"]  # test runs before build
```

Short-form tasks (just a command string) are also supported:

```toml
[tasks]
fmt  = "cargo fmt --check"
lint = "cargo clippy -- -D warnings"
```

### Lanes

Lanes compose tasks into pipelines:

```toml
[lanes.ci]
description = "Full CI pipeline"
fail_fast = true
steps = [
  { parallel = ["fmt", "lint"] },  # run concurrently
  "test",                            # sequential step
  "build",
]
```

### Parallel groups

Wrap multiple task names in `{ parallel = [...] }` to run them concurrently on separate threads:

```toml
steps = [
  { parallel = ["lint", "typecheck", "fmt"] },  # all run at the same time
  "test",                                          # waits for the group to finish
]
```

### Fail-fast

- `fail_fast = true` (default) — stop at first failure
- `fail_fast = false` — run all steps, collect and report all failures at the end

```toml
[lanes.audit]
description = "Surface all issues, not just the first one"
fail_fast = false
steps = ["security-check", "license-audit", "dependency-scan"]
```

### Inline commands

For one-off commands that don't need a named task:

```toml
steps = [
  { run = "echo Running CI..." },
  "test",
  { run = "echo Done!" },
]
```

### Task dependencies

Dependencies are declared on tasks, not lanes. When a lane runs a task, its deps run first automatically:

```toml
[tasks.publish]
cmd  = "cargo publish"
deps = ["build-release"]  # build-release runs before publish, always

[lanes.release]
steps = ["publish"]  # fledge resolves: build-release -> publish
```

## Contributing

PRs welcome for other languages or frameworks. Tag your repo with `fledge-lane` to make it discoverable via `fledge lane --search`.
