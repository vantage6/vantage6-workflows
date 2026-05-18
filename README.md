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
```

Replace `<commit-sha>` with a full commit SHA from this repository (pin updates when the reusable workflow changes).

The caller and this repository must live under the same GitHub organization (or otherwise satisfy GitHub’s [access rules for reusable workflows](https://docs.github.com/en/actions/using-workflows/reusing-workflows#access-to-reusable-workflows)).

### What it does

1. Parses the pushed tag into version components and resolves the remote branch the tag points at.
2. Installs dependencies with `uv sync` and reads `vantage6.common.__version__` for Docker base tagging.
3. Creates a GitHub Release (`softprops/action-gh-release`); prerelease when the tag has a non-numeric stage suffix.
4. Logs in to `ghcr.io` and runs `make image TAG=<tag> PUSH_REG=true VANTAGE6_VERSION=<resolved>`.

### Scope

Algorithm repos still on vantage6 v4 should keep their own workflows until they are
migrated to v5.