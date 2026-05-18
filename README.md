# vantage6-workflows

Reusable [GitHub Actions](https://docs.github.com/en/actions/using-workflows/reusing-workflows) for vantage6 projects.

## Algorithm release

Workflow: [`.github/workflows/algorithm-release.yml`](.github/workflows/algorithm-release.yml)

Intended for **vantage6** algorithm repositories that use `pyproject.toml`, **uv** (`uv sync`), Python **3.13**, and read the platform version from `vantage6.common.__version__`. It runs on a tag push in the **caller** repository (typically `*.*.*`).
In general, these requirements are met by algorithms created with the [v6-algorithm-template](https://github.com/vantage6/v6-algorithm-template) repository via the CLI command
``v6 algorithm create``.

Note that the workflow is designed for vantage6 v5+. It will not work for vantage6 v4
and older.

### Caller example

```yaml
name: Create Release

on:
  push:
    tags:
      - "*.*.*"

jobs:
  release:
    uses: vantage6/vantage6-workflows/.github/workflows/algorithm-release.yml@<commit-sha>
    secrets: inherit
    # Optional inputs / secrets (defaults match GHCR + GITHUB_TOKEN)
    # with:
    #   registry: docker.io
    #   registry_username: myuser
    #   docker_image: ghcr.io/myorg/algorithm/my-algo   # for v6 major-tag policy (see below)
    # secrets:
    #   registry_password: ${{ secrets.DOCKERHUB_TOKEN }}
```

Replace `<commit-sha>` with a full commit SHA from this repository (pin updates when the reusable workflow changes).

**Inputs**

| Name | Default | Purpose |
|------|---------|---------|
| `registry` | `ghcr.io` | Registry hostname for `docker/login-action` |
| `registry_username` | *(empty → `github.actor`)* (typical for GHCR) | Registry username |
| `docker_image` | *(empty)* | Full image name **without tag** (e.g. `ghcr.io/vantage6/algorithm/session-basics`). When set, algorithm **prereleases** still push the v6 major tag (`:<major>`) only if that tag does not exist in the registry yet; **stable** algorithm tags always allow the major tag. When empty, prereleases omit the major tag (callers that do not use `INCLUDE_V6_MAJOR_TAG` in `Makefile` are unaffected). |

**Secrets**

| Name | Required | Purpose |
|------|----------|---------|
| `registry_password` | No | Password or token for login; if omitted, `github.token` is used (typical for GHCR) |

With `secrets: inherit` only, `registry_password` is unset unless your repository defines a secret with the exact name `registry_password`. For a differently named PAT (e.g. `DOCKERHUB_TOKEN`), map it explicitly as in the example above.

The caller and this repository must live under the same GitHub organization (or otherwise satisfy GitHub’s [access rules for reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows#access-to-reusable-workflows)).

### What it does

1. Parses the pushed tag into version components and resolves the remote branch the tag points at.
2. Installs dependencies with `uv sync` and reads `vantage6.common.__version__` for Docker base tagging.
3. Creates a GitHub Release (`softprops/action-gh-release`); prerelease when the tag has a non-numeric stage suffix.
4. Logs in to the configured registry (defaults: `ghcr.io`, `github.actor`, `github.token`), decides whether to pass `INCLUDE_V6_MAJOR_TAG=true` to `make` when the `Makefile` supports it, and runs `make image TAG=<tag> PUSH_REG=true VANTAGE6_VERSION=<resolved>`.

### Scope

Algorithm repos still on vantage6 v4 should keep their own workflows until they are
migrated to v5.