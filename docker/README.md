# docker

Fledge lane for Docker image builds: lint Dockerfile, build, test the image, push to registry, and tag latest.

## Prerequisites

- [hadolint](https://github.com/hadolint/hadolint) for Dockerfile linting
- Docker daemon running with registry credentials configured
- Set `IMAGE_NAME` and `IMAGE_TAG` env vars (or override task env in your `fledge.local.toml`)

## Lanes

### `ci` — Build and test (no push)
Lints the Dockerfile, builds the image, and runs the container's test suite. Safe for PRs.

```bash
IMAGE_NAME=my-app IMAGE_TAG=pr-123 fledge run ci
```

### `publish` — Full build + push
Lint → build → test → push versioned tag → retag and push `latest`.

```bash
IMAGE_NAME=my-app IMAGE_TAG=1.2.3 fledge run publish
```

### `fast-build` — Parallel lint + layer cache warm-up
Runs hadolint and `docker pull` (for layer cache) in parallel, then builds. Cuts build time on cold CI runners.

```bash
fledge run fast-build
```

## Tasks

| Task | Command | Deps | Notes |
|------|---------|------|-------|
| `lint` | `hadolint Dockerfile` | — | Dockerfile best-practice linting |
| `build` | `docker build` | — | BuildKit enabled via env |
| `test` | `docker run … run-tests.sh` | `build` | Runs tests inside the built image |
| `push` | `docker push` | `build`, `test` | Pushes versioned tag |
| `push-latest` | `docker tag … && docker push` | `push` | Promotes to `latest` |

## Patterns demonstrated

- **Multiple task deps** — `push` depends on both `build` and `test`
- **Parallel + inline** — `fast-build` runs lint alongside an inline cache-warming command
- **Env vars in commands** — `$IMAGE_NAME` and `$IMAGE_TAG` interpolated at runtime
- **BuildKit** — `DOCKER_BUILDKIT = "1"` set in task env, not globally
