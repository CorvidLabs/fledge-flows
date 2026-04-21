# fledge-flows

A collection of example [fledge](https://github.com/CorvidLabs/fledge) flow configurations for common project types.

Each directory contains a `fledge.toml` with fully-documented tasks, flows, parallel groups, dependencies, and fail-fast settings.

## Available Flows

| Language | Path | Description |
|----------|------|-------------|
| Python | [`python/`](python/) | lint (ruff), format (ruff), typecheck (mypy), test (pytest) |
| Node/TypeScript | [`node-typescript/`](node-typescript/) | build (tsc), lint (eslint), format (prettier), test (vitest) |
| Rust | [`rust/`](rust/) | clippy, test, fmt, build, release |
| Go | [`go/`](go/) | vet, test, build, staticcheck |

## Importing Flows

### Search for flows

```bash
fledge flow --search
```

This searches GitHub for repos tagged with the `fledge-flow` topic and lists available flow collections.

### Import a specific flow

```bash
# Import by repo and path
fledge flow --import CorvidLabs/fledge-flows/rust

# Import into current project's fledge.toml
fledge flow --import CorvidLabs/fledge-flows/python
```

Imported flows are merged into your project's `fledge.toml`. Existing tasks/flows with the same name are not overwritten unless you pass `--force`.

### Manual import

Copy the relevant `fledge.toml` into your project and adjust commands to match your toolchain:

```bash
curl -sL https://raw.githubusercontent.com/CorvidLabs/fledge-flows/main/rust/fledge.toml \
  >> fledge.toml
```

## Flow Concepts

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

### Flows

Flows compose tasks into pipelines:

```toml
[flows.ci]
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
[flows.audit]
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

Dependencies are declared on tasks, not flows. When a flow runs a task, its deps run first automatically:

```toml
[tasks.publish]
cmd  = "cargo publish"
deps = ["build-release"]  # build-release runs before publish, always

[flows.release]
steps = ["publish"]  # fledge resolves: build-release -> publish
```

## Contributing

PRs welcome for other languages or frameworks. Tag your repo with `fledge-flow` to make it discoverable via `fledge flow --search`.
