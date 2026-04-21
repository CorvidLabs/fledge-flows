# release

Fledge flow for releasing a Node.js package: version bump, changelog generation, git tag, npm publish, and push.

## Flows

### `release` — Full release
Bumps patch version → generates changelog → commits → tags → publishes to npm → pushes with tags.

```bash
fledge run release
```

For minor or major bumps, override the version task:

```bash
npm version minor --no-git-tag-version && fledge run release
```

### `dry-run` — Preview without publishing
Runs version bump and changelog generation, then prints a summary. Nothing is committed or published.

```bash
fledge run dry-run
```

### `publish-only` — Re-publish existing tag
Skips version/changelog/commit steps. Use when a publish step failed mid-release.

```bash
fledge run publish-only
```

## Tasks

| Task | Command | Deps |
|------|---------|------|
| `version-bump` | `npm version patch --no-git-tag-version` | — |
| `changelog` | `conventional-changelog` | `version-bump` |
| `commit` | `git add -A && git commit` | `changelog` |
| `tag` | `git tag -a vX.Y.Z` | `commit` |
| `publish` | `npm publish` | `tag` |
| `push` | `git push --follow-tags` | `tag` |

## Patterns demonstrated

- **Inline run step** — `{ run = "echo '...'" }` for simple one-liners without defining a named task
- **Task deps chain** — each release step depends on the previous, enforcing strict ordering
- **Dry-run flow** — subset of tasks that terminates before any destructive/external action
- **Publish-only flow** — re-entrant partial flow for failure recovery
